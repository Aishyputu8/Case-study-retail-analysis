# Case-study-retail-analysis
/* A retail company uses data analytics to identify product performance, segment customers by purchase quantity, and analyze behavior to improve retention. With SQL for data cleaning and analysis, it aims to optimize inventory, enable targeted marketing, and enhance customer experience to counter stagnant growth and engagement decline. */

create database db;
**-- import tables to this db , schemas**
use ab;
select * from pcu; **-- CustomerID, Age, Gender ,Location , JoinDate**
select * from ppdt; **-- ProductID ,ProductName, Category, StockLevel, Price**
select * from psl; **-- TransactionID, CustomerID, ProductID ,QuantityPurchased ,TransactionDate ,Price**
desc pcu;
desc ppdt;
desc psl; 

alter table pcu 
change ï»¿CustomerID CustomerID int;
alter table ppdt 
rename column ï»¿ProductID to ProductID ;
alter table ppdt 
rename column ï»¿TransactionID to TransactionID ;

select count(*) from pcu; **-- 1000 customer**

select CustomerID, count(*) from pcu 
group by CustomerID 
having count(*) > 1; **-- no duplicate**

select ProductID, count(*) from ppdt 
group by ProductID 
having count(*) > 1; **-- no duplicate**

select TransactionID, count(*) from psl 
group by TransactionID 
having count(*) > 1; **-- 2 duplicate** - 4999, 5000

****/* remove deuplictaes from psl , remove deplicates wityhout changing its og table name*/****

create table psl1 as
select distinct * from psl; -- dummy table creation 

drop table psl; -- drop og one

alter table psl1
rename to psl; -- rename to og one

