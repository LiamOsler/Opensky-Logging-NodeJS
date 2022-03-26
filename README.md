# Opensky-Logging-NodeJS

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
