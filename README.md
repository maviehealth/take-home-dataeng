# P1 DataEngineer take home task

You have been hired as a data engineer consultant for our company.

Our company's objective is to enhance the operations of doctor's offices through analytics. To achieve this, we need to collect data from the offices and make it accessible to the business team, who will then devise a plan to optimize each office's performance.

Your objective is to equip the business team with a centralized database that holds data from a single doctor's office. Specifically, you need to build a data pipeline that ingests data generated by the offices into the database and compute relevant statistics.

We count on you, as the success of our company heavily relies on the quality of the data we gather and analyze :)

## Prerequisites

1. Install docker: [[linux]](https://docs.docker.com/desktop/install/linux-install/#supported-platforms) [[mac]](https://docs.docker.com/desktop/install/mac-install/)
2. Install docker compose plugin : [[linux]](https://docs.docker.com/compose/install/linux/) (for mac its already in docker)

## Task
This project simulates a publish-subscriber setup [[wiki]](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern). The doctor's offices generate data (publisher) and your code will read and process those data (subscriber).

The publisher is generated by us, and we provide a lot of helper code for the subscriber :)

**Setup:**

Our project involves 3 entities:
* *Publishers*: Two streams of data generating an object each every 2 seconds.
* *Subscribers*: Two listeners of those streams that trigger a function on the arrival of every new object
* *Database*: A postgres database tha must store all new objects

Your task is to:
1. Use our subscriber library to push the new data to postgres
2. Extract some analytics using sql on top of those data

**Know your data**

There are three tables in our database: `Patient`, `Claim` and `Diagnose`:
* `Patient` contain information like `first_name`, `last_name`, etc. of patients in the office
* `Claim` stores information about the "bills" payed by each patient when visiting the office, like:
  * `id`: The id of the claim. **THIS SHOULD BE ALWAYS UNIQUE**
  * `code`: The type of billing (in other words, type of examination)
  * `price`: The price of the claim
  * `patient_id`: The id of a patient
  * `tsp`: The timestamp it was created
* `Diagnose` stores information about the diagnosed disease of the patients like:
  * `id`: The id of the diagnose. **THIS SHOULD BE ALWAYS UNIQUE**
  * `icd10_code`: A code showing the type of the medical condition
  * `patient_id`: The id of a patient
  * `tsp`: The timestamp it was diagnosed

The `Patient` table is filled by us with 1000 random people. The last two tables are empty, and you need to fill them with data read from the *publishers*.

*Publishers* produce `claims` and `diagnoses` every 2 seconds in JSON format, like the following:
```
Claim: {'id': 0, 'patient_id': 67, 'code': '03230', 'price': 3}
Diagnose: {'id': 0, 'patient_id': 744, 'icd10_code': 'R41.3'}
```

*NOTE:* The three aforementioned tables are pre-define in sql and created when [running](#how-to-run) this project with the help of [init_db.sql](setup/init_db.sql) script.


### Task 1: Data Ingest

We have set-up a python main function as a scaffold in file [ingestor.py](src/ingestor.py) with comments to walk you through.

**Scenario:** Our library code handles subscribing to `claim` an `diagnose` publishers and can call a function every time there are new data. Write your own functions to ingest those data in postgres whenever they arrive. **BUT BE CAREFUL**  duplicate data may be pushed through the pipeline. `Claim` and `diagnose` data are perceived as duplicate when their `id`(which is also a primary key in their respective sql tables) has been used before in the past. Duplicate data should be discarded.

**Your objectives:**
1. Create a connection to postgres (database configs are provided as global vars in main, use `psycopg2` library)
2. Write (a) function(s) that will push `claim` and `diagnose` data in the respective postgres tables
3. Ignore duplicate data whenever they appear


### Task 2: Data Process

We have set-up a python main function as a scaffold in file [analytics.py](src/analytics.py) with comments to walk you through.

**Scenario:** Now that your ingestion works and data flow in the database we need to create a new table that will store some analytics for our business team. They need to know how the total revenue that is generated from patients with 'Amnesia' (icd10 code `R41.3`) and 'Asthma' (icd10 code `J44.9`) every 10 minutes per category (Amnesia, Asthma).

**Your objectives**
1. Design the table that will store the analytics described above
2. Create a connection to postgres
3. Run the query to create the table you designed in step 1
4. Design the query that will generate the analytics requested above
5. Use our `scheduler` library to schedule your query (from step 4) to run every 10 minutes


## How to run

```bash
docker-compose build # once
...
docker-compose up # whenever you want to run the pipeline. To stop use CTRL^C.
                  # You can re-run the pipeline just by using `docker-compose up` again
...
docker-compose down # When you are finished with all tasks
```

The `docker-compose up` will spin up:
* A postgres database with credentials
  * `user=dataeng`, `password=best`, `host=db`, `port=5432`, `database_name=doctor_office`
* A bare minimum SQL adminer that you can use to see the tables of the database and do queries
  * Just open up `localhost:8080` and login with the above credentials (check that database type is Postgres and not MySQL)
* The publisher, subscriber (Taks 1)
* The analytics post processor (Task 2)

## General guidelines

1. Feel free to add files and classes in the project, but whatever you add should under the [./src](./src) folder
2. You need to write code inside [ingestor.py](src/ingestor.py) and [analytics.py](src/analytics.py) (and in any new file you may create)
3. Some code parts in the files above are listed as `DON'T REMOVE THIS`, please don't remove those :P. Apart from that you are free to add anything you see fit.
4. Please don't alter the code in: [publisher.py](src/publisher.py), [pipeline_utils.py](src/utils/pipeline_utils.py), [scheduler.py](src/models/scheduler.py) and [subscriber.py](src/models/subscriber.py)
5. Have fun :)

## Questions

For any questions feel free to contact me whenever you want by:
* email: belmpas.theofilos [at] praxis-eins.de

