# Case-study-retail-analysis
/* A retail company uses data analytics to identify product performance, segment customers by purchase quantity, and analyze behavior to improve retention. With SQL for data cleaning and analysis, it aims to optimize inventory, enable targeted marketing, and enhance customer experience to counter stagnant growth and engagement decline. */

create database db;
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
having count(*) > 1; **-- 2 duplicate** 

****/* remove deuplictaes from psl , remove deplicates wityhout changing its og table name*/****

create table psl1 as
select distinct * from psl; 
drop table psl; 
alter table psl1
rename to psl; 

****/* how each table is related / connected to each other*/****
select * from psl s
left join ppdt p on s.ProductID = p.ProductID
where p.ProductID is not null;

**/*check if there is any mismatch / discrepency in joins*/**
set Price = (
		select p.price from ppdt p
        where p.ProductID = s.ProductID )
where s.ProductID in (
		select ProductID from ppdt where s.ProductID <> p.ProductID);

****/* missing value , null*/****
select count(*) from pcu where location is null ;
select count(*), gender , location from pcu  where location = '' group by location , gender;
update pcu -- rplace mv with words 
set location = 'unknown'
where location is null or location ='';

**/*date */****
select * from psl;
set JoinDate = str_to_date(JoinDate, '%d/%m/%y');
update psl
set TransactionDate = str_to_date(TransactionDate, '%d/%m/%y');

