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

**#EDA**
****/* Distribution of customer_profiles Table , location based */****
				select count(CustomerID), Location 
				from pcu
				group by Location
				order by count(CustomerID) desc;

****/* Distribution of customer_profiles Table , location, gender based */****
				select count(CustomerID) , Gender , Location
				from pcu
				group by Gender, Location
				order by count(CustomerID) desc;

/* **yearly / monthly / user** */
				select count(CustomerID) as newuser, year(JoinDate) as year
                from pcu
                group by year(JoinDate);

				select count(CustomerID) as newuser , month(JoinDate) as month
                from pcu
                group by  month(JoinDate);
                
  				select count(CustomerID) as newuser , day(JoinDate) as day
                from pcu
                group by  day(JoinDate);        
                
/* **double entry** */
					Select ProductName, 
					count(*) from  ppdt 
					group by ProductName having count(*) > 1 ;
                    
						Select Category, 
						count(ProductName) as count_product,
						sum(StockLevel) as sum_stocklevel,
						round(avg(StockLevel),2) as avg_stocklevel,
						round(avg(Price),2) as Avg_Price
					from ppdt
					Group by Category
					ORDER BY sum_stocklevel DESC;

/***total sale transaction***/
					select ProductID,
                    sum(QuantityPurchased * price) as sales
                    from psl 
                    group by ProductID ;
                    
/***Get a summary of total sales and quantities sold per product.***/
					select ProductID,
                    sum(QuantityPurchased * price) as sales,
                    sum(QuantityPurchased)
                    from psl 
                    group by ProductID ;
 
       
					Select 
						pi.ProductName,
						st.ProductID, 
						sum(st.QuantityPurchased*st.Price) as TotalSales, 
						sum(st.QuantityPurchased)
					from psl st
					JOIN ppdt pi on pi.ProductID = st.ProductID
					Group by st.ProductID, pi.ProductName;                   
				Select 
				pi.Category,
					round(sum(st.QuantityPurchased*st.Price),2) as TotalSales, 
                    round(Avg(st.Price),2) as Avg_price,
					sum(st.QuantityPurchased)
				from psl st
				JOIN ppdt pi on pi.ProductID = st.ProductID
				Group by pi.Category order by Avg_price desc;  

/* **1. total sales , qntity catg based** */                    
						Select 
							pi.Category ,
							round(sum(st.QuantityPurchased*st.Price),2) as TotalSales, 
							sum(st.QuantityPurchased),
							round(Avg(st.Price),2) as Avg_price
						from psl st
						JOIN ppdt pi on pi.ProductID = st.ProductID
						Group by pi.Category 
						ORDER BY TotalSales DESC; 

/* **2. Customer Purchase Frequency***/                        
				Select 
					CustomerID, 
					count(*) as total_transitions    
				from psl
				Group by CustomerID
				ORDER by  total_transitions DESC;
						
 /* **3. High Sales Products  top 10 -- top 5***/
 
				select ProductID, round(sum(QuantityPurchased * Price),2) as sales
                from psl
                group by ProductID order by sales desc limit 5;
				
/* **4. Low Sales Products***/
				select ProductID, round(sum(QuantityPurchased * Price),2) as sales
                from psl
                group by ProductID order by sales limit 5;							

/* **5. Sales Trends with rank** */                    
						with ab as (select  year(TransactionDate) as year,
                        month(TransactionDate) as month,
                        	round(sum(QuantityPurchased*Price),2) as TotalSales,
							count(*) as total_trans
                        from psl group by year, month)
                        select year, month,TotalSales , total_trans , 
                        dense_rank() over(order by year , month)  as ranks from ab;
                    
/* **6. Growth rate of sales M-o_M  Write a SQL query to understand the month on month growth rate of sales of the company 
which will help understand the growth trend of the company.** */
				with ab as (
                select extract(month from TransactionDate) as month,
                round(sum(QuantityPurchased*Price),2) as sales from psl
                group by extract(month from TransactionDate))
                
					select month, sales, 
                    lag(sales) over(order by month) as prev,
                    ((sales -  lag(sales) over(order by month)) / lag(sales) over(order by month)) *100 as mom
                    from ab order by month;
                    
			select month, 
				lag(sales) over(order by month) as prev,
				((sales -  lag(sales) over(order by month)) / lag(sales) over(order by month)) *100 as mom
			from (
			select extract(month from TransactionDate) as month , 
                round(sum(QuantityPurchased*Price),2) as sales 
                from psl
                group by extract(month from TransactionDate)
			) as mosale order by month;
  
/* **12. Customers - High Purchase Frequency and Revenue***/
		select CustomerID, count(*) as freq,
        round(sum(QuantityPurchased*Price),2) as totalsales
         from psl
         group by CustomerID 
	  having freq >10  and totalsales > 1000
         order by freq desc;

