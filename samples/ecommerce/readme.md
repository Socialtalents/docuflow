# Seed database for an abstract ecommerce app

Seed admin user:

```
./docuflow  export mongodb://127.0.0.1:27017/ecommerce -c users
>>
1761942040045113700.import.users-0.json
```

Export indexes for users so we will have index on login field:

```
./docuflow  export mongodb://127.0.0.1:27017/ecommerce -c users --indexes
>>
1761942148908424400.ensure-indexes.users.json
```

Inital set of categories:
```
./docuflow  export mongodb://127.0.0.1:27017/ecommerce -c categories
>>
1761942148908744400.import.categories-0.json
```

Inital set of warehouses:
```
./docuflow  export mongodb://127.0.0.1:27017/ecommerce -c warehouses
>>
1761942148908744401.import.warehouses-0.json
```

Text index for categories to demonstrate full text search:

```
./docuflow  export mongodb://127.0.0.1:27017/ecommerce -c categories --indexes
>>>
1761942957881993000.ensure-indexes.categories.json
```

Geo index for categories to demonstrate full text search:
```
./docuflow  export mongodb://127.0.0.1:27017/ecommerce -c warehouses --indexes
>>>
1761944629540342600.ensure-indexes.warehouses.json
```
