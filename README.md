# Opensky-Logging-NodeJS

Logs all Callsigns reported on Opensky every 10 seconds using the OpenSky REST API.

```js
const cron = require('node-cron');
const axios = require('axios');
const fs = require('fs')

cron.schedule('*/10 * * * * *', () => {
    console.log('hello');
    axios
        .get('https://opensky-network.org/api/states/all')
        .then(res => {
            console.log(`statusCode: ${res.status}`)
            console.log(res.data.time)
            console.log(res.data);
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


OpenSky:
Matthias Schäfer, Martin Strohmeier, Vincent Lenders, Ivan Martinovic and Matthias Wilhelm.
"Bringing Up OpenSky: A Large-scale ADS-B Sensor Network for Research".
In Proceedings of the 13th IEEE/ACM International Symposium on Information Processing in Sensor Networks (IPSN), pages 83-94, April 2014.

The OpenSky Network, https://opensky-network.org

