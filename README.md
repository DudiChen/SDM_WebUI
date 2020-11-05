POST
http://localhost:8080/SDM/api/login
Request Body:
{
    "userName": ${userName},
    "password": ${password}
}

POST
http://localhost:8080/SDM/api/signup
Request Body:
{
    "userName": ${userName},
    "password": ${password},
    "role": ${role}
}


[Consumer]
GET
http://localhost:8080/SDM/api/areas

[Consumer]
GET
http://localhost:8080/SDM/api/users

[Consumer]
GET
http://localhost:8080/SDM/api/balance?uuid=${loginResponse.uuid}

GET
http:localhost:8080/SDM/api/users/transactions?uuid=${loginResponse.uuid}

POST
http://localhost:8080/SDM/api/users/transactions?uuid=${loginResponse.uuid}
Request Body:
{
    "amount": ${transactionAmount},
    "date": ${transactionDate}
}

GET
http://localhost:8080/SDM/api/areas/stores?areaId=${selectedArea.id}

GET
http://localhost:8080/SDM/api/areas/products?areaId=${selectedArea.id}

GET
http://localhost:8080/SDM/api/areas/stores/products?areaId=${selectedArea.id}&storeId=${selectedStore.id}

GET
http://localhost:8080/SDM/api/areas/stores/products/discounts?areaId=${selectedArea.id}&storeId=${selectedStore.id}&productId=${selectedProductInStore.id}

[Consumer]
POST
http://localhost:8080/SDM/api/areas/stores/orders?areaId=${selectedArea.id}&storeId=${selectedStore.id}
{
    "order": ${selectedProducts},
    "discounts": ${selectedDiscounts}
}

