# Labs: Type Conversions

For this lab, use the following file : [sensor_data.json](./files/sensor_data.json)

If you analyze schema with MongoDB Compass, you can see that `count` field has several data types :

- string
- double
- int32
- undefined

Question : What is the sum total of the count information?

```js
db.data.aggregate([
    {
        "$unwind": "$turnstile"
    },
    {
        "$group": {
            "_id": 0,
            "sum": {
                "$sum": {
                    $convert:{
                        input: "$turnstile.count",
                        to: "int",
                        onNull: "",
                        onError: 0
                    }
                }
            }
        }
  }
])
```

Answer: 281245