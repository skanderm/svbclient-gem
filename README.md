# svbclient Ruby gem

A gem which you can use in your Ruby / Ruby on Rails app to simplify
HMAC-signing and headers.

It also includes helpers for interacting with the Accounts, ACH, Book Transfer, Wire Transfer, and Onboarding API endpoints.

## Install

On the command line:

```
gem install svbclient
```

In a Gemfile:

```ruby
gem 'svbclient', '~> 3.2.3'
```

## Direct API access

```ruby
require 'svbclient'
require 'json'

client = SVBClient.new(API_KEY, HMAC_SECRET)

# full client options
SVBClient.new(API_KEY, HMAC_SECRET, base_url: 'https://api.svb.com')

response = client.get("/v1", "query=test")

# you receive the full response from the Rest-Client gem
# https://github.com/rest-client/rest-client
# here's how you get the data:
response_data = JSON.parse(response.body)
# { "data": { "address_line1": "", ... } }
core_response = JSON.parse(response.body)["data"]

# create
client.post("/v1", { "body": "content" })

# update
client.patch("/v1/companies/:id", { "body": "content" })

# delete
client.delete("/v1/companies/:id")

# uploading a file (png, jpg, or pdf)
client.upload('/v1/files', File.new('./images/test.png'), 'image/png')
```

## Accounts API helpers

```ruby
require 'svbclient'

client = SVBClient.new(API_KEY, HMAC_SECRET)
my_account = SVBClient::Account.new(client, account_id)
# returns SVBClient::Account object

# request all accounts as SVBClient::Account objects
SVBClient::Account.all(client)

# poll the server for up-to-date data on this record
my_account.data

# poll the server for transactions
response = my_account.transactions
response["transactions"] # array
response["links"] # paging

# filter transactions on this account
my_account.transactions(start_date: 'YYYY-MM-DD', end_date: 'YYYY-MM-DD')
```

## ACH API helpers

```ruby
client = SVBClient.new(API_KEY, HMAC_SECRET)
handler = SVBClient::ACHHandler.new(client)

my_ach = handler.create({ account_number: '', ... })
# returns SVBClient::ACH object

# poll the server for up-to-date data on this record
my_ach.data
my_ach.data["status"]

# cancel an ACH
# you cannot set any other status
my_ach.update_status('canceled')

# return a list of filtered ACHs (one or both filters)
handler.find({ status: 'pending', effective_date: '2017-05-30' })
```

## Book Transfer API helpers

```ruby
client = SVBClient.new(API_KEY, HMAC_SECRET)
handler = SVBClient::BookTransferHandler.new(client)

my_transfer = handler.create({ account_number: '', ... })
# returns SVBClient::BookTransfer object

# poll the server for up-to-date data on this record
my_transfer.data
my_transfer.data["status"]

# cancel a book transfer
# you cannot set any other status
my_transfer.update_status('canceled')
```

## Wire Transfer API helpers

```ruby
client = SVBClient.new(API_KEY, HMAC_SECRET)
handler = SVBClient::WireHandler.new(client)

my_wire = handler.create({ account_number: '', ... })
# returns SVBClient::ACH object

# poll the server for up-to-date data on this record
my_wire.data
my_wire.data["status"]

# cancel a pending wire transfer
# you cannot set any other status
my_wire.update_status('canceled')
```

## Onboarding API helpers

```ruby
client = SVBClient.new(api_key, hmac)
Onboarding = SVBClient::Onboarding.new(client)

# create an address
address = Onboarding.address(street_line1: '221B Baker St', city: 'London', country: 'GB')
# address is a new SVBClient::Onboarding::Address object
puts address.id

# update the address
address.update({ street_line1: '222 Baker St' })

# poll the server for up-to-date data on this record
address.data

# retrieve an old address record
old_address = Onboarding.address(id: 101)
old_address.data
# { "street_line1": "", ... }

# delete
old_address.delete

# a reference to an address, address -> address_id
person = Onboarding.person(address: address, first_name: '', ...)

# file upload
file = Onboarding.file(file: File.new('test.png'), mimetype: 'image/png')
```
