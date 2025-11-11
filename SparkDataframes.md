## Create single column dataframe using list
creating a list:
ages_list  = [21,22,23,24]

type(age_list) returns "list"

You can run help(spark.createDataframe) > gives you information on what you are able to create
so to create the spark dataframe you have to pass the type INT 
so writing: spark.createDataframe(Ages_list) fails because you have to specify the data type
spark.createDataframe(ages_list, 'int') > so you can pass the type by a string like this and this runs successfully
spark.createDataframe(ages_list, IntegerType) is another way to do this 
But for that to run first, you have to run:
from pyspark.sql.types import IntegerType
then run: spark.createDataframe(Ages_list, IntegerType()) > this is successful

you can make another list of text like: names_list = ['Scott','Valerie']
and then run: spark.createDataframe(names_list, 'string')
or you can also import a string type first:from pyspark.sql.types import StringType
then you can run:spark.createDataframe(names_list, StringType())


## Create multiple column dataframe using list
First, we create a list of tuples. You put parenthesees for each tuple
ages_list = [ (21,),(23,),(24,)]

if you run: type(ages_list) it returns list
if we run type(Ages_list[2[) it returns tuple

spark.createDataframe(ages_list) > this runs succesfully 
so if you have a list of tuples like this we dont have to specify the schema
if you run:spark.createDataframe(ages_list, 'int') fails because we are using tuples. If you want to specify the schema, you specify the column name and data type like this:
spark.createDataframe(ages_list, 'age int')

this is how you can create a single column dataframe using list of tuples where each tuple only has one element

now we can create a multi column df with each touple containing one or more columns:

user_lists = [(1,'scott'),(2,'valerie)] this is the user id and name, the datatypes are integer and string 
spark.createDataframe(users_list) > this returns type bigint,string
or we can specify the type for each column by:
spark.createDataframe(users_list,'id int','name string') 

## Overview of Spark Row
First, we will start with a list and dataframe:

```
user_lists = [(1,'scott'),(2,'valerie)]
spark.createDataframe(users_list,'id int','name string') 
df.show 
```

spark dataframe is a collection of row objects. you can run:
```
df.collect()
```

collect will convert the dataframe to a python list. You can also run:
```
type(df.collect())
```
 > returns list 
<img width="219" height="133" alt="{E7ECBBBE-A6B1-4816-A03A-28440A5C134F}" src="https://github.com/user-attachments/assets/8c6c9dd6-b7ee-4407-9ce9-3c860832c89a" />

You can import row by running:
```
from pyspark.sql import Row

help(Row)
```
> its a class that takes in 2 arguments 
The row object represents a single row in a dataframe

## Convert list of lists into Spark dataframe using row
```
user_lists = [(1,'scott'),(2,'valerie)]
spark.createDataframe(users_list,'id int','name string') 
df.show 
```
Now we can convert this list of lists to a list of rows by using list comprehensions or a function:
we use * because once we have the list you should be able to convert to varying arguments:
```
from pyspark import Row
users_rows = [Row(*user) for user in users_list]
users_row -- will display the output for each row, with type row
```
each element of this list is of type row. to create the dataframe now you can run:

```
spark.createDataframe(user_rows)
```
why are we using *user?
lets use this:
def dummy(*args):
print(args)
dummy(1) > shows the argument as (1,) it printed as a tuple
if you add another argument like:
dummy(1,'hello') will shows us both
sometimes when we use functions like dummy, which accepts varying argments, we might want to pass a list and we would like to process each element in the list as a separate element by interperting the argument as tuples

if we pass the list directly:
user_details = [1,'Scott']
if you run dummy(user_details) its considering it as one argument, not as 2 
sometimes we have to convert the list into different arguments rather than passing as one arguemnt. so if we add a star:
dummy(user_details) we now return 2 arguments, as opposed to the first time we ran it:

<img width="157" height="84" alt="{B2815F38-7ACF-41CB-B2FB-AF14F78AEE61}" src="https://github.com/user-attachments/assets/b1cfb5fd-0b44-4c29-a65a-252c11eb6720" />

you can also print(len(args)) and you can see without the * we get 1, and with * we get 2:

<img width="135" height="104" alt="{73F267D8-FD7C-4598-ABB0-024927A143AE}" src="https://github.com/user-attachments/assets/83aa5235-f44c-46fe-8846-9147113924b9" />

## Convert list of tuples into spark dataframe using row 
