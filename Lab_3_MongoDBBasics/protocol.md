
### 1)	Створіть декілька товарів з різним набором властивостей. 
```
db.product.insert([
    {
        category:"Phone",
        model:"iPhone 6",
        producer:"Apple",
        price:600
    }])

db.product.insert([
    {
        category:"Monitors",
        model:"HP Pavilion 21.5-Inch IPS LED HDMI VGA",
        producer:"HP",
        size: 21.5,
        price:90
    }])

db.product.insert([
    {
        category:"Monitors",
        model:"HP 23.8-inch FHD IPS",
        producer:"HP",
        size: 23.8,
        price:130
    }])

db.product.insert([
    {
        category:"Monitors",
        model:"Samsung CHG90 Series Curved 49",
        producer:"Samsung",
        size:49.0,
        price:1039
    }])
```
### 2)	Напишіть запит, який виведіть усі товари (відображення у JSON)
```
db.product.find().pretty()
```
###### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a466c70d7b8fe5bff298450"),
    "category" : "Phone",
    "model" : "iPhone 6",
    "producer" : "Apple",
    "price" : 600.0
}

/* 2 */
{
    "_id" : ObjectId("5a466db4d7b8fe5bff298451"),
    "category" : "Monitors",
    "model" : "HP Pavilion 21.5-Inch IPS LED HDMI VGA",
    "producer" : "HP",
    "size" : 21.5,
    "price" : 90.0
}

/* 3 */
{
    "_id" : ObjectId("5a466ddbd7b8fe5bff298452"),
    "category" : "Monitors",
    "model" : "HP 23.8-inch FHD IPS",
    "producer" : "HP",
    "size" : 23.8,
    "price" : 130.0
}

/* 4 */
{
    "_id" : ObjectId("5a466e0ed7b8fe5bff298453"),
    "category" : "Monitors",
    "model" : "Acer R240HY bidx 23.8-Inch IPS HDMI DVI VGA",
    "producer" : "Acer",
    "size" : 23.8,
    "price" : 135.0
}

/* 5 */
{
    "_id" : ObjectId("5a466e49d7b8fe5bff298454"),
    "category" : "Monitors",
    "model" : "Dell Computer Ultrasharp U2415 24.0",
    "producer" : "Dell",
    "size" : 24.0,
    "price" : 215.0
}

/* 6 */
{
    "_id" : ObjectId("5a466e6ad7b8fe5bff298455"),
    "category" : "Monitors",
    "model" : "Dell UltraSharp U2414H 23.8",
    "producer" : "Dell",
    "size" : 23.8,
    "price" : 188.0
}

/* 7 */
{
    "_id" : ObjectId("5a466e86d7b8fe5bff298456"),
    "category" : "Monitors",
    "model" : "Dell UltraSharp U2715H 27",
    "producer" : "Dell",
    "size" : 27.0,
    "price" : 326.0
}

/* 8 */
{
    "_id" : ObjectId("5a466ea8d7b8fe5bff298457"),
    "category" : "Monitors",
    "model" : "Samsung CHG90 Series Curved 49",
    "producer" : "Samsung",
    "size" : 49.0,
    "price" : 1039.0
}
```
### 3)	Підрахуйте скільки товарів певної категорії
```
db.product.aggregate(
  [
    {
      $match: {
        category: {
          $eq: "Monitors"
        }
      }
    },
    {
      $count: "number of product in Monitors category"
    }
  ]
)
```
###### responce:
```
/* 1 */
{
    "number of product in Monitors category" : 7
}
```
### 4)	Напишіть запити, які вибирають товари за різними критеріям і їх сукупності: категорія та ціна (в проміжку), розмір (наприклад розмір взуття або діагональ екрану) або модель, конструкція з використанням in
```
db.product.find( { 
    producer: { $in: [ "Dell", "Samsung" ] },
    size: {$gt: 25}
    } )
```
###### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a466e86d7b8fe5bff298456"),
    "category" : "Monitors",
    "model" : "Dell UltraSharp U2715H 27",
    "producer" : "Dell",
    "size" : 27.0,
    "price" : 326.0
}

/* 2 */
{
    "_id" : ObjectId("5a466ea8d7b8fe5bff298457"),
    "category" : "Monitors",
    "model" : "Samsung CHG90 Series Curved 49",
    "producer" : "Samsung",
    "size" : 49.0,
    "price" : 1039.0
}
```
### 5)	Виведіть список всіх виробників товарів без повторів
```
db.product.distinct("producer")
```
##### responce:
```
/* 1 */
[
    "Apple",
    "HP",
    "Acer",
    "Dell",
    "Samsung"
]
```
### 6)	Оновить певні товари, змінивши існуючі значення і додайте нові властивості (характеристики) товару за певним критерієм
```
db.product.update({  
   "_id":ObjectId("5a466c70d7b8fe5bff298450")
},
{  
   $set:{  
      price:400,
      status:"Discontinued",

   }
})
```
##### let's check:
```
db.product.findOne({  
   "_id":ObjectId("5a466c70d7b8fe5bff298450")
})
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a466c70d7b8fe5bff298450"),
    "category" : "Phone",
    "model" : "iPhone 6",
    "producer" : "Apple",
    "price" : 400.0,
    "status" : "Discontinued"
}
```
### 7)	Знайдіть товари у яких є (присутнє поле) певні властивості
```
db.product.find( { status: { $exists: true } } )
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a466c70d7b8fe5bff298450"),
    "category" : "Phone",
    "model" : "iPhone 6",
    "producer" : "Apple",
    "price" : 400.0,
    "status" : "Discontinued"
}
```
### 1)	Створіть кілька замовлень з різними наборами товарів, але так щоб один з товарів був у декількох замовленнях
```
db.order.insert([  
   {  
      "order_number":2017122901,
      "date":ISODate("2017-12-29"),
      "total_sum":690,
      "customer":{  
         "name":"Ivan",
         "surname":"Ivanov",
         "phones":[  
            9876543,
            1234567
         ],
         "address":"Khreshchatyk 1, Kyiv, UA"
      },
      "payment":{  
         "card_owner":"Ivan Ivanov",
         "cardId":123456789
      },
      "order_items_id":[  
         {  
            "$ref":"product",
            "$id":ObjectId("5a466c70d7b8fe5bff298450")
         },
         {  
            "$ref":"product",
            "$id":ObjectId("5a466db4d7b8fe5bff298451")
         }
      ]
   },
   {  
      "order_number":2017122902,
      "date":ISODate("2017-12-29"),
      "total_sum":530,
      "customer":{  
         "name":"Peter",
         "surname":"Petrov",
         "phones":[  
            3456789,
            7654321
         ],
         "address":"Khreshchatyk 2, Kyiv, UA"
      },
      "payment":{  
         "card_owner":"Peter Petrov",
         "cardId":987654321
      },
      "order_items_id":[  
         {  
            "$ref":"product",
            "$id":ObjectId("5a466c70d7b8fe5bff298450")
         },
         {  
            "$ref":"product",
            "$id":ObjectId("5a466ddbd7b8fe5bff298452")
         }
      ]
   }
])
```
### 2)	Виведіть всі замовлення
```
db.order.find()
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 690.0,
    "customer" : {
        "name" : "Ivan",
        "surname" : "Ivanov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298459"),
    "order_number" : 2017122902.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 530.0,
    "customer" : {
        "name" : "Peter",
        "surname" : "Petrov",
        "phones" : [ 
            3456789.0, 
            7654321.0
        ],
        "address" : "Khreshchatyk 2, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Peter Petrov",
        "cardId" : 987654321.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }
    ]
}
```
### 3)	Виведіть замовлення з вартістю більше певного значення
```
db.order.find( { total_sum: { $gt: 600 } } )
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 690.0,
    "customer" : {
        "name" : "Ivan",
        "surname" : "Ivanov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }
    ]
}
```
### 4)	Знайдіть замовлення зроблені одним замовником
```
db.order.find({  
   "customer.name":"Ivan",
   "customer.surname":"Ivanov",
})
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 690.0,
    "customer" : {
        "name" : "Ivan",
        "surname" : "Ivanov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }
    ]
}
```
### 5)	Знайдіть всі замовлення з певним товаром (товарами) (шукати можна по ObjectId)
```
db.order.find({'order_items_id.$id': ObjectId("5a466c70d7b8fe5bff298450")})
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 905.0,
    "customer" : {
        "name" : "Ivan",
        "surname" : "Ivanov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298459"),
    "order_number" : 2017122902.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 530.0,
    "customer" : {
        "name" : "Peter",
        "surname" : "Petrov",
        "phones" : [ 
            3456789.0, 
            7654321.0
        ],
        "address" : "Khreshchatyk 2, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Peter Petrov",
        "cardId" : 987654321.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }
    ]
}
```
### 6)	Додайте в усі замовлення з певним товаром ще один товар і збільште існуючу вартість замовлення на Х
```
db.order.update({  

},
{  
   $inc:{  
      "total_sum":215
   },
   $addToSet:{  
      "order_items_id":{  
         "$ref":"product",
         "$id":ObjectId("5a466e49d7b8fe5bff298454")
      },

   }
},
{  
   arrayFilters:[  
      {  
         "order_items_id.$id":ObjectId("5a466c70d7b8fe5bff298450")
      }
   ], "multi": true
})
```

##### let's check:
```
db.order.find({'order_items_id.$id': ObjectId("5a466c70d7b8fe5bff298450")})
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 1120.0,
    "customer" : {
        "name" : "Ivan",
        "surname" : "Ivanov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298459"),
    "order_number" : 2017122902.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 745.0,
    "customer" : {
        "name" : "Peter",
        "surname" : "Petrov",
        "phones" : [ 
            3456789.0, 
            7654321.0
        ],
        "address" : "Khreshchatyk 2, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Peter Petrov",
        "cardId" : 987654321.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}
```
### 7)	Виведіть кількість товарів в певному замовленні
```
db.order.aggregate({  
   $unwind:"$order_items_id"
},
{  
   $match:{  
      "_id":ObjectId("5a469f0dd7b8fe5bff298458")
   }
},
{  
   $group:{  
      _id:null, count:{$sum:1}
   }
})
```
##### responce:
```
/* 1 */
{
    "_id" : null,
    "count" : 3.0
}
```

### 8)	Виведіть тільки власників і номери кредитних карт, вартість замовлень яких перевищує певну суму
```
db.order.find( { total_sum: { $gt: 1000 } }, {"_id": 0, "payment.card_owner": 1, "payment.cardId": 1 } )
```
##### responce:
```
/* 1 */
{
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    }
}
```
### 9)	Видаліть товар з замовлень, зроблених за певний період дат

##### first, find all:
```
db.order.find()
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-28T00:00:00.000Z"),
    "total_sum" : 1120.0,
    "customer" : {
        "name" : "Ivan",
        "surname" : "Ivanov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298459"),
    "order_number" : 2017122902.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 745.0,
    "customer" : {
        "name" : "Peter",
        "surname" : "Petrov",
        "phones" : [ 
            3456789.0, 
            7654321.0
        ],
        "address" : "Khreshchatyk 2, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Peter Petrov",
        "cardId" : 987654321.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}

/* 3 */
{
    "_id" : ObjectId("5a46b58dd7b8fe5bff29845a"),
    "order_number" : 2017122701.0,
    "date" : ISODate("2017-12-27T00:00:00.000Z"),
    "total_sum" : 265.0,
    "customer" : {
        "name" : "John",
        "surname" : "Smith",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "MAIN STREET 1, LA, USA"
    },
    "payment" : {
        "card_owner" : "John Smith",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e0ed7b8fe5bff298453")
        }
    ]
}

/* 4 */
{
    "_id" : ObjectId("5a46b5efd7b8fe5bff29845b"),
    "order_number" : 2017122601.0,
    "date" : ISODate("2017-12-27T00:00:00.000Z"),
    "total_sum" : 130.0,
    "customer" : {
        "name" : "John",
        "surname" : "Smith",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "MAIN STREET 1, LA, USA"
    },
    "payment" : {
        "card_owner" : "John Smith",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }
    ]
}
```

##### Видаляємо товар з замовлень між 27 (включно) та 28:
```
db.order.update({  
   date:{  
      $gte: ISODate("2017-12-27T00:00:00.000      Z"),
      $lt:  ISODate("2017-12-28T00:00:00.000      Z")
   }
},
{  
   $pull:{"order_items_id":{}}
},
false,
true)
```
##### let's check:
```
db.order.find()
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-28T00:00:00.000Z"),
    "total_sum" : 1120.0,
    "customer" : {
        "name" : "Ivan",
        "surname" : "Ivanov",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298459"),
    "order_number" : 2017122902.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 745.0,
    "customer" : {
        "name" : "Peter",
        "surname" : "Petrov",
        "phones" : [ 
            3456789.0, 
            7654321.0
        ],
        "address" : "Khreshchatyk 2, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Peter Petrov",
        "cardId" : 987654321.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}

/* 3 */
{
    "_id" : ObjectId("5a46b58dd7b8fe5bff29845a"),
    "order_number" : 2017122701.0,
    "date" : ISODate("2017-12-27T00:00:00.000Z"),
    "total_sum" : 265.0,
    "customer" : {
        "name" : "John",
        "surname" : "Smith",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "MAIN STREET 1, LA, USA"
    },
    "payment" : {
        "card_owner" : "John Smith",
        "cardId" : 123456789.0
    },
    "order_items_id" : []
}

/* 4 */
{
    "_id" : ObjectId("5a46b5efd7b8fe5bff29845b"),
    "order_number" : 2017122601.0,
    "date" : ISODate("2017-12-27T00:00:00.000Z"),
    "total_sum" : 130.0,
    "customer" : {
        "name" : "John",
        "surname" : "Smith",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "MAIN STREET 1, LA, USA"
    },
    "payment" : {
        "card_owner" : "John Smith",
        "cardId" : 123456789.0
    },
    "order_items_id" : []
}
```
### 10)	Перейменуйте у всіх замовлення ім'я (прізвище) замовника
```
db.order.update({  
   'customer.name':{  
      $exists:true
   },
   'customer.surname':{  
      $exists:true
   }
},
{  
   $set:{  
      'customer.name':'Chuck',
      'customer.surname':'Noris'
   }
},
{  
   multi:true
})
```
##### let's check:
```
db.order.find()
```
##### responce:
```
/* 1 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298458"),
    "order_number" : 2017122901.0,
    "date" : ISODate("2017-12-28T00:00:00.000Z"),
    "total_sum" : 1120.0,
    "customer" : {
        "name" : "Chuck",
        "surname" : "Noris",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "Khreshchatyk 1, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Ivan Ivanov",
        "cardId" : 123456789.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466db4d7b8fe5bff298451")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}

/* 2 */
{
    "_id" : ObjectId("5a469f0dd7b8fe5bff298459"),
    "order_number" : 2017122902.0,
    "date" : ISODate("2017-12-29T00:00:00.000Z"),
    "total_sum" : 745.0,
    "customer" : {
        "name" : "Chuck",
        "surname" : "Noris",
        "phones" : [ 
            3456789.0, 
            7654321.0
        ],
        "address" : "Khreshchatyk 2, Kyiv, UA"
    },
    "payment" : {
        "card_owner" : "Peter Petrov",
        "cardId" : 987654321.0
    },
    "order_items_id" : [ 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466c70d7b8fe5bff298450")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466ddbd7b8fe5bff298452")
        }, 
        {
            "$ref" : "product",
            "$id" : ObjectId("5a466e49d7b8fe5bff298454")
        }
    ]
}

/* 3 */
{
    "_id" : ObjectId("5a46b58dd7b8fe5bff29845a"),
    "order_number" : 2017122701.0,
    "date" : ISODate("2017-12-27T00:00:00.000Z"),
    "total_sum" : 265.0,
    "customer" : {
        "name" : "Chuck",
        "surname" : "Noris",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "MAIN STREET 1, LA, USA"
    },
    "payment" : {
        "card_owner" : "John Smith",
        "cardId" : 123456789.0
    },
    "order_items_id" : []
}

/* 4 */
{
    "_id" : ObjectId("5a46b5efd7b8fe5bff29845b"),
    "order_number" : 2017122601.0,
    "date" : ISODate("2017-12-27T00:00:00.000Z"),
    "total_sum" : 130.0,
    "customer" : {
        "name" : "Chuck",
        "surname" : "Noris",
        "phones" : [ 
            9876543.0, 
            1234567.0
        ],
        "address" : "MAIN STREET 1, LA, USA"
    },
    "payment" : {
        "card_owner" : "John Smith",
        "cardId" : 123456789.0
    },
    "order_items_id" : []
}
```
