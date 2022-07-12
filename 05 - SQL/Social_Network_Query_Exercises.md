# SQL Social Network Query Exercises Answers

These are my solutions to the SQL Social Network Query exercises

The database can be viewed here: https://courses.edx.org/asset-v1:StanfordOnline+SOE.YDB-SQL0001+2T2020+type@asset+block/socialdata.html


## Q1

Find the names of all students who are friends with someone named Gabriel. 

```sql
select name
from Highschooler, (select ID2 from Friend where ID1 in (select ID from Highschooler where name = 'Gabriel'))
where Highschooler.ID = ID2;
```

## Q2

For every student who likes someone 2 or more grades younger than themselves, 
return that student's name and grade, and the name and grade of the student they like. 

```sql
select H1.name, H1.grade, H2.name, H2.grade
from Likes, Highschooler H1, Highschooler H2
where Likes.ID1 = H1.ID and Likes.ID2 = H2.ID and (H1.grade - H2.grade >= 2);
```

## Q3

For every pair of students who both like each other, return the name and grade of both 
students. Include each pair only once, with the two names in alphabetical order. 

```sql
select H1.name, H1.grade, H2.name, H2.grade
from Highschooler H1, Highschooler H2, (select L1.ID1, L1.ID2 from Likes L1, Likes L2 where L1.ID1 = L2.ID2 and L1.ID2 = L2.ID1) L
where L.ID1 = H1.ID and L.ID2 = H2.ID and H1.name < H2.name;
```

## Q4

Find all students who do not appear in the Likes table (as a student who likes or is 
liked) and return their names and grades. Sort by grade, then by name within each grade. 

```sql
select name, grade
from Highschooler
where ID not in (select Likes.ID1 as likes from Likes union select Likes.ID2 as likes from Likes)
order by grade, name;
```

## Q5

For every situation where student A likes student B, but we have no information about 
whom B likes that is, B does not appear as an ID1 in the Likes table, return A and B's 
names and grades.


```sql
select H1.name, H1.grade, H2.name, H2.grade
from Highschooler H1,  Highschooler H2, (select ID1, ID2 from Likes where ID2 not in (select ID1 from Likes)) L
where L.ID1 = H1.ID and L.ID2 = H2.ID;
```

## Q6

Find names and grades of students who only have friends in the same grade. Return the result 
sorted by grade, then by name within each grade. 


```sql
select H.name, H.grade from Highschooler as H,
        (select F.ID1 as ID from Friend as F
            except
         select F.ID1
            from Highschooler as H1, Highschooler as H2, Friend as F
            where H1.grade <> H2.grade and F.ID1 = H1.ID and 
            F.ID2 = H2.ID) as T
    where H.ID = T.ID
    order by H.grade, H.name;
```

## Q7

For each student A who likes a student B where the two are not friends, find if they have a 
friend C in common (who can introduce them!). For all such trios, return the name and grade 
of A, B, and C. 

```sql
select H1.name, H1.grade, H2.name, H2.grade, H3.name, H3.grade 
    from Highschooler as H1, Highschooler as H2, Highschooler as H3, 
    Friend as F1, Friend as F2,   
    -- pairs A, B who are not friends
    (select L.ID1, L.ID2 from Likes as L
    except
    select L.ID1, L.ID2
    from Likes as L, Friend as F
    where L.ID1 = F.ID1 and L.ID2 = F.ID2) as T1
    
    where H1.ID = F1.ID1 and F1.ID1 = T1.ID1 and 
          H2.ID = F2.ID1 and F2.ID1 = T1.ID2 and 
          H3.ID = F2.ID2 and F2.ID2 = F1.ID2;
```

## Q8


Find the difference between the number of students in the school and the number of different 
first names. 

```sql
select abs(A.c-B.c) from
    (select count(*) as c from (select distinct H.name from Highschooler as H)) as A, 
    (select count(*) as c from Highschooler) as B;
```


## Q9

Find the name and grade of all students who are liked by more than one other student. 

```sql
select H.name, H.grade
    from Highschooler as H, 
        (select L.ID2 as ID from Likes as L
                        group by L.ID2
                        having count(L.ID2) > 1) as T
    where T.ID = H.ID;
```
