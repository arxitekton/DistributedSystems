### Змоделювати наступну предметну область:
-	Є: Items, Customers, Orders
-	Customer може додати Item(s) до Order (тобто купити Товар)
-	У Customer може бути багато Orders
-	Item може входити в багато Orders
-	Customer може переглядати (view), але при цьому не купувати Items 

```
CREATE CONSTRAINT ON (i:Item) ASSERT i.sku IS UNIQUE;
CREATE CONSTRAINT ON (c:Customer) ASSERT c.email IS UNIQUE;
CREATE CONSTRAINT ON (o:Order) ASSERT o.no IS UNIQUE;

CREATE (item1:Item { name : "Notebook Asus zzz", sku: "1234567", price: 1000 })
CREATE (item2:Item { name : "Mouse Logitech yyy", sku: "1234568", price: 20 })
CREATE (item3:Item { name : "Notebook HP www", sku: "1234569", price: 1100 })
CREATE (item4:Item { name : "Mouse HP xxx", sku: "1234570", price: 25 })
CREATE (item5:Item { name : "Notebook Sony Vaio", sku: "1234571", price: 900 })
CREATE (item6:Item { name : "Mouse cH xxx", sku: "1234572", price: 15 })

CREATE (customer1:Customer { name : "Smith", email: "sm@gmail.com" })
CREATE (customer2:Customer { name : "Ivanoff", email: "iv@gmail.com" })
CREATE (customer3:Customer { name : "Petroff", email: "pf@gmail.com" })

CREATE (order1:Order { no:'#20180220001', date: "2018-02-20" })
CREATE (order2:Order { no:'#20180220002', date: "2018-02-20" })
CREATE (order3:Order { no:'#20180221003', date: "2018-02-21" })
CREATE (order4:Order { no:'#20180222004', date: "2018-02-22" })

CREATE (item1)-[:WAS_VIEWED_BY]->(customer1)
CREATE (item2)-[:WAS_VIEWED_BY]->(customer1)
CREATE (customer1)-[:CREATED]->(order1)
CREATE (customer1)-[:ORDERED]->(item1)-[:IN_ORDER]->(order1)
CREATE (customer1)-[:ORDERED]->(item2)-[:IN_ORDER]->(order1)

CREATE (item1)-[:WAS_VIEWED_BY]->(customer2)
CREATE (item2)-[:WAS_VIEWED_BY]->(customer2)
CREATE (customer2)-[:CREATED]->(order2)
CREATE (customer2)-[:ORDERED]->(item1)-[:IN_ORDER]->(order2)
CREATE (customer2)-[:ORDERED]->(item2)-[:IN_ORDER]->(order2)

CREATE (item1)-[:WAS_VIEWED_BY]->(customer3)
CREATE (item2)-[:WAS_VIEWED_BY]->(customer3)
CREATE (item3)-[:WAS_VIEWED_BY]->(customer3)
CREATE (item4)-[:WAS_VIEWED_BY]->(customer3)
CREATE (item5)-[:WAS_VIEWED_BY]->(customer3)
CREATE (item6)-[:WAS_VIEWED_BY]->(customer3)
CREATE (customer3)-[:CREATED]->(order3)
CREATE (customer3)-[:ORDERED]->(item3)-[:IN_ORDER]->(order3)
CREATE (customer3)-[:ORDERED]->(item4)-[:IN_ORDER]->(order3)
CREATE (customer3)-[:CREATED]->(order4)
CREATE (customer3)-[:ORDERED]->(item1)-[:IN_ORDER]->(order4)
CREATE (customer3)-[:ORDERED]->(item2)-[:IN_ORDER]->(order4)
RETURN *
```

### Знайти Items які входять в конкретний Order
```
MATCH (items:Item)-[:IN_ORDER]->(order:Order{ no: "#20180220001"}) RETURN items
```
###### responce:
```
items
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
{
  "name": "Notebook Asus zzz",
  "sku": "1234567",
  "price": 1000
}
```
### Знайти всі Orders конкретного Customer
```
MATCH (orders:Order)<-[:CREATED]-(customer:Customer{ email: "sm@gmail.com" }) RETURN orders
```
###### responce:
```
orders
{
  "date": "2018-02-20",
  "no": "#20180220001"
}
```
### Знайти всі Items куплені конкретним Customer
```
MATCH (items:Item)<-[:ORDERED]-(customer:Customer{ email: "sm@gmail.com" }) RETURN items
```
###### responce:
```
items
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
{
  "name": "Notebook Asus zzz",
  "sku": "1234567",
  "price": 1000
}
```
### Знайти кількість Items куплені конкретним Customer
```
MATCH (items:Item)<-[:ORDERED]-(customer:Customer{ email: "sm@gmail.com" }) RETURN count(items)
```
###### responce:
```
count(items)
2
```
### Знайти всі Items переглянуті (view) конкретним Customer
```
MATCH (items:Item)-[:WAS_VIEWED_BY]->(customer:Customer{ email: "pf@gmail.com" }) RETURN items
```
###### responce:
```
items
{
  "name": "Notebook Sony Vaio",
  "sku": "1234571",
  "price": 900
}
{
  "name": "Mouse cH xxx",
  "sku": "1234572",
  "price": 15
}
{
  "name": "Notebook HP www",
  "sku": "1234569",
  "price": 1100
}
{
  "name": "Mouse HP xxx",
  "sku": "1234570",
  "price": 25
}
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
{
  "name": "Notebook Asus zzz",
  "sku": "1234567",
  "price": 1000
}
```
### Знайти всі Items переглянуті (view), але не куплені конкретним Customer
```
MATCH (items:Item)-[:WAS_VIEWED_BY]->(customer:Customer{ email: "pf@gmail.com" }) WHERE NOT (items)<-[:ORDERED]->(customer) RETURN items
```
###### responce:
```
items
{
  "name": "Notebook Sony Vaio",
  "sku": "1234571",
  "price": 900
}
{
  "name": "Mouse cH xxx",
  "sku": "1234572",
  "price": 15
}
```
### Знайти Items що куплені разом з конкретним Item (тобто все Items що входять до Order разом з даними Item)
```
MATCH (items:Item)-[:IN_ORDER]->(order:Order)<-[:IN_ORDER]-(item:Item{ sku: "1234567" }) WHERE items.sku <> item.sku  RETURN order.no, items  ORDER BY order.no
```
###### responce:
```
order.no	items
"#20180220001"	
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
"#20180220002"	
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
"#20180222004"	
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
```
### Знайти Customers які купили даний конкретний Item
```
MATCH (customers:Customer)-[:ORDERED]->(item:Item{ sku: "1234567" }) RETURN customers
```
###### responce:
```
customers
{
  "name": "Petroff",
  "email": "pf@gmail.com"
}
{
  "name": "Ivanoff",
  "email": "iv@gmail.com"
}
{
  "name": "Smith",
  "email": "sm@gmail.com"
}
```
```
MATCH (customers:Customer)-[:ORDERED]->(item:Item{ sku: "1234570" }) RETURN customers
```
###### responce:
```
customers
{
  "name": "Petroff",
  "email": "pf@gmail.com"
}
```
### Знайти всі Items куплені Customer(s) які купили даний конкретний Item
```
MATCH (items:Item)<-[:ORDERED]-(customers:Customer)-[:ORDERED]->(item:Item{ sku: "1234570" }) RETURN items
```
###### responce:
```
items
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
{
  "name": "Notebook Asus zzz",
  "sku": "1234567",
  "price": 1000
}
{
  "name": "Notebook HP www",
  "sku": "1234569",
  "price": 1100
}
```
### Знайти всі Items куплені Customer(s) який переглядав даний конкретний Item
```
MATCH (items:Item)<-[:ORDERED]-(customers:Customer)<-[:WAS_VIEWED_BY]-(item:Item{ sku: "1234571" }) RETURN items
```
###### responce:
```
items
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
{
  "name": "Notebook Asus zzz",
  "sku": "1234567",
  "price": 1000
}
{
  "name": "Mouse HP xxx",
  "sku": "1234570",
  "price": 25
}
{
  "name": "Notebook HP www",
  "sku": "1234569",
  "price": 1100
}
```

### Реалізувати у вигляді составного запиту рекомендаційну систему, яка за певним конкретному Item знаходить список Items, які:
-	були куплені разом з цим Item
-	були куплені Customer(s) який купив цей Item
-	були переглянуті Customer(s) які купили цей Item

```
MATCH (items:Item)<-[:ORDERED|:WAS_VIEWED_BY]->(customers:Customer)-[:ORDERED]->(item:Item{ sku: "1234570" }) WHERE items.sku<>item.sku RETURN DISTINCT items
```
###### responce:
```
items
{
  "name": "Notebook Asus zzz",
  "sku": "1234567",
  "price": 1000
}
{
  "name": "Mouse Logitech yyy",
  "sku": "1234568",
  "price": 20
}
{
  "name": "Notebook HP www",
  "sku": "1234569",
  "price": 1100
}
{
  "name": "Notebook Sony Vaio",
  "sku": "1234571",
  "price": 900
}
{
  "name": "Mouse cH xxx",
  "sku": "1234572",
  "price": 15
}
```