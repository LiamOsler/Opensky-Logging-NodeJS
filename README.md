# Opensky-Logging-NodeJS

Logging use NodeJS and the OpenSky REST API.

## Requirements:
Via ``npm``:
- axios
- express
- fs
- node-cron

axios:
- used to make http requests to 'https://opensky-network.org/api/states/all', the OpenSky REST API.

express:
- serves the stored data and index.html over http

fs:
- for reading and writing to the file system

node-cron:
- schedules the http requests


I host these files locally in the ```public/scripts/``` folder as they're only used by the frontend viewer application 


### server.js:

Writing the the logs to a file name based on the unix datetime with a ```.txt``` file extension (this will create a new file in the ```public/data/all/``` directory every time the ```cron-job``` runs:
```js
const cron = require('node-cron');
const axios = require('axios');
const fs = require('fs')

//Set the interval at which the requests are made with cron.schedule, in this example, every ten seconds:
cron.schedule('*/10 * * * * *', () => {
    axios
        .get('https://opensky-network.org/api/states/all')
        .then(res => {
            let fileString = 'public/data/'+res.data.time.toString()+'.txt';
            fs.writeFile(fileString, JSON.stringify(res.data), err => {
                if (err) {
                    console.error(err)
                    return
                }
            });
        })
        .catch(error => {
            console.error(error)
        })
});
```

For convenience, we can additionally create a log called ```latest.txt``` which is continuously updated with the most recently reported log every time the cron job runs:
```js
cron.schedule('*/20 * * * * *', () => {
    axios
        .get('https://opensky-network.org/api/states/all')
        .then(res => {
            let time = res.data.time;
            let fileString = 'public/data/all/'+time+'.txt';
            let latest = 'public/data/all/latest.txt';

            fs.writeFile(fileString, JSON.stringify(res.data), err => {
                if (err) {
                    console.error(err);
                    return;
                }
            });
            fs.writeFile(latest, JSON.stringify(res.data), err => {
                if (err) {
                    console.error(err);
                    return;
                }
            });
        })
        .catch(error => {
            console.error(error);
        })
});
```

Retrieving the latest log using setInterval (in this example, the latest log is requested every 1000 ms, on the client side:
```js
setInterval(function () {
    fetch('data/all/latest.txt')
        .then(response => response.json())
        .then(data => 
            aircraftLocations = data
        );
}, 1000);
```


## Logging individual aircraft:
We can make some additions to the server.js file if we want to log each individual aircraft's previous track:
```js
cron.schedule('*/20 * * * * *', () => {
    axios
        .get('https://opensky-network.org/api/states/all')
        .then(res => {
            let time = res.data.time;
            let fileString = 'public/data/all/'+time+'.txt';
            let latest = 'public/data/all/latest.txt';

            fs.writeFile(latest, JSON.stringify(res.data), err => {
                if (err) {
                    console.error(err);
                    return;
                }
            });

            for(let item of res.data.states){
                let dir = 'public/data/aircraft/' + item[0] +'/';
                if (!fs.existsSync(dir)){
                    fs.mkdirSync(dir);
                }
                let fileString = 'public/data/aircraft/'+item[0] +'/' +item[4]+'.txt';
                let latest = 'public/data/aircraft/'+item[0] +'/ latest.txt';
                fs.writeFile(fileString, JSON.stringify(item), err => {
                    if (err) {
                        console.error(err);
                        return;
                    }
                });
                fs.writeFile(latest, JSON.stringify(item), err => {
                    if (err) {
                        console.error(err);
                        return;
                    }
                });
            }

        })
        .catch(error => {
            console.error(error);
        })
});
```
Additionally, we may want to check that the file structure for the data folders exists using ```fs.existsSync``` before the cron scheduled job:
```
if (!fs.existsSync('public/data/all/')){
    fs.mkdirSync('public/data/all/');
}
if (!fs.existsSync('public/data/aircraft/')){
    fs.mkdirSync('public/data/aircraft/');
}
```

We also want to serve these files, we can do this easily with express, everything in the ```public``` (particularly ```public/data/```, where the logged data is stored) directory will be served on port 3000:
```
const app = express();
const port = 3000;

app.use(express.static('public'))

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`);
})
```
Additionally, we can add our index.html file to the ```public``` folder, which will function as the clientside access point for the application:

With each of these elements all together we have this in the ```server.js``` file:

```js
const express = require('express');
const cron = require('node-cron');
const axios = require('axios');
const fs = require('fs')

if (!fs.existsSync('public/data/all/')){
    fs.mkdirSync('public/data/all/');
}
if (!fs.existsSync('public/data/aircraft/')){
    fs.mkdirSync('public/data/aircraft/');
}

cron.schedule('*/20 * * * * *', () => {
    axios
        .get('https://opensky-network.org/api/states/all')
        .then(res => {
            let time = res.data.time;
            let fileString = 'public/data/all/'+time+'.txt';
            let latest = 'public/data/all/latest.txt';


            fs.writeFile(latest, JSON.stringify(res.data), err => {
                if (err) {
                    console.error(err);
                    return;
                }
            });

            for(let item of res.data.states){
                let dir = 'public/data/aircraft/' + item[0] +'/';
                if (!fs.existsSync(dir)){
                    fs.mkdirSync(dir);
                }
                let fileString = 'public/data/aircraft/'+item[0] +'/' +item[4]+'.txt';
                let latest = 'public/data/aircraft/'+item[0] +'/ latest.txt';
                fs.writeFile(fileString, JSON.stringify(item), err => {
                    if (err) {
                        console.error(err);
                        return;
                    }
                });
                fs.writeFile(latest, JSON.stringify(item), err => {
                    if (err) {
                        console.error(err);
                        return;
                    }
                });
            }

        })
        .catch(error => {
            console.error(error);
        })
});


const app = express();
const port = 3000;

app.use(express.static('public'))

app.listen(port, () => {
  console.log(`Flights3D listening on port ${port}`);
})
```


OpenSky:
Matthias Schäfer, Martin Strohmeier, Vincent Lenders, Ivan Martinovic and Matthias Wilhelm.
"Bringing Up OpenSky: A Large-scale ADS-B Sensor Network for Research".
In Proceedings of the 13th IEEE/ACM International Symposium on Information Processing in Sensor Networks (IPSN), pages 83-94, April 2014.

The OpenSky Network, https://opensky-network.org

