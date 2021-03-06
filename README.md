Bank
==============

This is a simple Bank server written in Ruby on Rails (v.4.0.4).

### Prepration and Execution
Ensure you are running a Postgres database server.
The database settings are in config/database.yml

To prepare the sample data, execute 
```
rake db:reset
```

Install the gems:
```
bundle install
```

Start the server
```
rails server
```

### Interfaces

#### Web-Client
*****
##### Authentication
The admin interface can be accessed by visiting the root of the site and providing a username of "admin@example.com" and a password of "password".

The only user-facing interface is provided by ActiveAdmin.  This interface provides CRUD access to the models:
- Admin User
- User
- Account
- Transaction Log

#### API
*****
##### Authentication
The API is protected via the presense of 2 headers in any API request:
X-Card-Number and X-Pin - these values are validated in a filter on the AccountController.  If the CardNumber and Pin aren't valid, a 403 is returned from the API requests.

##### User:
A GET to /api/user with a valid X-Card-Number and X-Pin will return the user's information with rel links in addition to any accounts that the user has, also with rel links for 'self', 'withdraw' and 'balance'
```
{
  card_number: "123123123"
  pin: "1234"
  links: [{
    type: "self"
    uri: http://localhost:3000/api/user
  }]
  accounts: 
  [{
    id: 1
    title: "Checking"
    balance: "8789.87"
    links: [{
      type: "self"
      uri: http://localhost:3000/api/accounts/1
    }{
      type: "withdraw"
      uri: http://localhost:3000/api/accounts/1/withdraw
    }{
      type: "balance"
      uri: http://localhost:3000/api/accounts/1/balance
    }]
   }{
    id: 5
    title: "Tax Free Savings"
    balance: "0.0"
    links: [{
      type: "self"
      uri: http://localhost:3000/api/accounts/5
    }{
      type: "withdraw"
      uri: http://localhost:3000/api/accounts/5/withdraw
    }{
      type: "balance"
      uri: http://localhost:3000/api/accounts/5/balance
    }]
  }]
}
```
##### Accounts:
A GET to /api/accounts/{:id} (a rel 'self' link in the above results) returns the following:
```
[{
  id: 1
  title: "Checking"
  balance: "8789.87"
  links: [{
    type: "self"
    uri: http://localhost:3000/api/accounts/1
  }{
    type: "withdraw"
    uri: http://localhost:3000/api/accounts/1/withdraw
  }{
    type: "balance"
    uri: http://localhost:3000/api/accounts/1/balance
  }]
}{
  id: 5
  title: "Tax Free Savings"
  balance: "0.0"
  links: [{
    type: "self"
    uri: http://localhost:3000/api/accounts/5
  }{
    type: "withdraw"
    uri: http://localhost:3000/api/accounts/5/withdraw
  }{
    type: "balance"
    uri: http://localhost:3000/api/accounts/5/balance
  }]
}]
```

##### Withdraw: 
A POST to `/api/account/{id}/withdraw` passing a param of 'amount'
The response will be something like:
```
{
  "response":
    {
      "result":false,
      "error":"Insufficient balance for this request.  Available balance: 9992.63",
      "balance":"9992.63"
    }
}
```
When the balance of the account is not sufficient for the withdraw request.

This is the response when there is a sufficient balance:
```
{
  "response":
    {
      "result":true,
      "error":null,
      "balance":"9992.63"
    }
}
```

##### Balance Inquiry
A GET to `/api/account/1/balance`
Response:
```
{
  "account":
  {
    "balance":"9892.4"
  }
}
```

### Audit Logging
When a balance is changed on an account an audit log entry is created with a record of the Account, User, Balance before the transaction, the Balance after the transaction and a date stamp.  These audit logs are found in the Admin Interface in the Transaction Log menu item.

### TODO
Unit testing on the API requests don't exist - was having a hard time finding how to do the tests I want.
