# Case-study-retail-analysis
/* A retail company uses data analytics to identify product performance, segment customers by purchase quantity, and analyze behavior to improve retention. With SQL for data cleaning and analysis, it aims to optimize inventory, enable targeted marketing, and enhance customer experience to counter stagnant growth and engagement decline. */

create database db;
-- import tables to this db , schemas
use ab;
select * from pcu;
 -- CustomerID, Age, Gender ,Location , JoinDate
select * from ppdt; -- ProductID ,ProductName, Category, StockLevel, Price
select * from psl; -- TransactionID, CustomerID, ProductID ,QuantityPurchased ,TransactionDate ,Price
desc pcu;
desc ppdt;
desc psl; 

alter table pcu 
change ï»¿CustomerID CustomerID int;
alter table ppdt 
rename column ï»¿ProductID to ProductID ;
alter table ppdt 
rename column ï»¿TransactionID to TransactionID ;
