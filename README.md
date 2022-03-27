# Opensky-Logging-NodeJS

Logs all Callsigns reported on Opensky every 10 seconds using the OpenSky REST API.

### server.js:

Writing the the logs to a file name based on the unix datetime with a ```.txt``` file extension (this will create a new file witha  :
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

For convenience, create a log called "latest" which is continuously updated with the most recently reported log:
```

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

Retrieving the latest log using setInterval (in this example, the latest log is requested every 1000 ms:
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



OpenSky:
Matthias Schäfer, Martin Strohmeier, Vincent Lenders, Ivan Martinovic and Matthias Wilhelm.
"Bringing Up OpenSky: A Large-scale ADS-B Sensor Network for Research".
In Proceedings of the 13th IEEE/ACM International Symposium on Information Processing in Sensor Networks (IPSN), pages 83-94, April 2014.

The OpenSky Network, https://opensky-network.org

