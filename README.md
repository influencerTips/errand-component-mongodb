# errand-mongodb
> [errand](https://github.com/errandjs/errand) worker component used for mongodb

## Usage

```

npm install errand-mongodb

```

Notes:

1. For dependencies and suggested usage of errand worker components refer to [errand](https://github.com/errandjs/errand)
2. Set environment variable ERRAND_MONGODB_URL with connection string for mongodb server, if not set module will default to `mongodb://localhost:27017`

## Example

```

{
	"tasks": [

		{
			"task": "errand-mongodb",
			"data": {
				"description": "replace-with-task-description",
				"request": {
					"method": "replace-with-mongodb-method",
					"parameters": {
						...
					}
				}
			}
		}

	]
}

```

Notes:

* **tasks** - [errand](https://github.com/errandjs/errand) task list
* **tasks[].task** - required `errand-mongodb` task name
* **tasks[].data.description** - optional task description
* **tasks[].data.request.method** - required mongodb method
* **tasks[].data.request.parameters** - required mongodb method parameters, the parameter payload will vary depending on method

### db.collection.aggregate Example

```

{
	"tasks": [
		{
			"task": "errand-mongodb",
			"data": {
				"description": "replace-with-task-description",
				"request": {
					"method": "db.collection.aggregate",
					"parameters": {
						"database": "errand_test",
						"collection": "orders",
						"pipeline": [
							{ "$match": { "status": "A" } },
							{ "$group": { "_id": "$cust_id", "total": { "$sum": "$amount" } } },
							{ "$sort": { "total": -1 } },
							{ "$out": "errand_aggregate_test_result" }
						],
						"helpers": [
							{ "created_at": "lastday" }
						]
					}
				}
			}
		}
	]
}

```

Notes:

* **tasks[].data.request.parameters.database** - mongodb database name
* **tasks[].data.request.parameters.collection** - mongodb collection name
* **tasks[].data.request.parameters.pipeline** - for pipeline source data refer to [group by and calculate a sum example in mongodb db.collection.aggregate() documentation](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate). Note that in this case with `$out` collection `errand_aggregate_test_result` will be replaced with result. [replaceKeywords function](#replacekeywords-function) is applied to all pipeline.
* **tasks[].data.request.parameters.helpers** - used to add helpers to beginning of aggregate pipeline where, each object consists of `key` and `value` where key contains name of field to apply function in value. Helper functions include:
  * **lastday** - is used to add date range for matching records from the previous day.
	* **lastweek** - is used to add date range for matching records from the previous week.

### db.collection.createIndex Example


```

	{
		"tasks": [
			{
				"task": "errand-mongodb",
				"data": {
					"description": "replace-with-task-description",
					"request": {
						"method": "db.collection.createIndex",
						"parameters": {
							"database": "errand_test",
							"collection": "orders",
							"keys": { ... },
							"options": { ... }
						}
					}
				}
			}
		]
	}

```

Notes:

* Refer to [db.collection.createIndex()](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/)
* **tasks[].data.request.parameters.database** - mongodb database name
* **tasks[].data.request.parameters.collection** - mongodb collection name
* **tasks[].data.request.parameters.keys** - db.collection.createIndex() keys
* **tasks[].data.request.parameters.options** - db.collection.createIndex() options

## replaceKeywords function

The replaceKeywords function is used to replace keywords with variables. Date based keywords replace the entire variable.

### replaceKeywords keywords

* `{{_date_minus1_startOf}}` replaced with start of yesterday, or `new Date(moment().add(-1, 'days').startOf('day')))`
* `{{_date_minus8_startOf}}` replaced with start of 8 days ago, or `new Date(moment().add(-8, 'days').startOf('day')))`
* `{{_date_minus1_endOf}}` replaced with end of yesterday, or `new Date(moment().add(-1, 'days').endOf('day')))`
