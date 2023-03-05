```
PUT /products

GET /_cat/nodes?v

GET /_cat/indices?v

GET /products/_search
{
    "query": {
        "match_all": {}
    }
}

GET /products/_search
{
    "query": {
        "match_all": {}
    }
}

POST /products/_doc
{
    "name": "Coffee Maker",
    "price": 64,
    "in_stock": 10
}

# create product with specific id
PUT /products/_doc/100
{
    "name": "Toaster",
    "price": 64,
    "in_stock": 10
}

# Retrieve documents by id
GET /products/_doc/100

# update product
POST /products/_update/100
{
    "doc": {
        "in_stock": 3
    }
}

# add fields to existing documents
POST /products/_update/100
{
    "doc": {
        "tags": ["electronics"]
    }
}


# Update in_stock property
# script is the object we need to use scripted updates
# source is where we put our script
# ctx is short for context. It is related to the context of the document
# _source is the property of the document
# -- substracts the value by 1
# ++ increments the value by 1
# = 1 set the value as 1
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock--"
    }
}

# We access the "params" object to get the quantity we want to subtract from in_stock property
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock -= params.quantity",
        "params": {
            "quantity": 4
        }
    }
}

# updates the "in_stock" property only if its value is bigger than 0
POST /products/_update/100
{
    "script": {
        "source": """
            if (ctx._source.in_stock == 0) {
                ctx.op = 'noop';
            }

            ctx._source.in_stock--;
        """
    }
}

# Upsert
POST /products/_update/100
{
    "script": {
        "source": "ctx._source.in_stock++"
    },
    "upsert": {
        "name": "Blender",
        "price": 399,
        "in_stock": 5
    }
}

# Replaces the document with ID 100
# If any property is missing in the new document it will be lost
PUT /products/_doc/100
{
    "name": "Toaster",
    "price": 49,
    "in_stock": 4
}

DELETE /products/_doc/100

# update all products in_stock property, similar to Update Where in RDBMS
POST /products/_update_by_query
{
    "script":{
        "source":"ctx._source.in_stock--"
    },
    "query":{
        "match_all":{}
    }
}

# delete all products that match the query
POST /products/_delete_by_query
{
    "query":{
        "match_all":{}
    }
}

# not specified index in the endpoint
# 1- define the action (index, create, update, delete). The action should be simply defined as a key within the object. The value for the key should be a JSON object
# 2- Create an index named products and add a document with _id 200. If the _id is not specified it will be generated automatically by Elasticsearch
# 3- Define the fields and values for the document at the line below. The keys are the fields and the values are the field's values
# The first line defines the action and some metadata, while the second line defines the source document
# The difference between the index and create actions is that the create action will fail if the document already exists. The 'index' action will add a new document, otherwise it will be replaced
POST /_bulk
{"index": {"_index": "products", "_id": 200}}
{"name": "Espresso Machine", "price": 199, "in_stock": 5}
{"create": {"_index": "products", "_id": 201}}
{"name": "Milk Frother", "price": 149, "in_stock": 14}

# We can use the index name in the endpoint if all the actions are for the same index
# The delete action is the only one that doesn't expect a second line
POST /products/_bulk
{"update": {"_id": 201}}
{"doc": {"price": 129 }}
{"delete": {"_id": 200}}
```