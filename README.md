# SQL_country_club_financial_analysis
/* Welcome to the SQL country club analysis project. 

PART 1: PHPMyAdmin
You will complete questions 1-9 below in the PHPMyAdmin interface. 

The data you need is in the "country_club" database. This database
contains 3 tables:
    i) the "Bookings" table,
    ii) the "Facilities" table, and
    iii) the "Members" table.

Using these tables, answer the following questions. */


/* QUESTIONS 
/* Q1: Some of the facilities charge a fee to members, but some do not.
Write a SQL query to produce a list of the names of the facilities that do. */

SELECT name
FROM `Facilities`
WHERE membercost >0

/* Q2: How many facilities do not charge a fee to members? */

SELECT COUNT( * )
FROM `Facilities`
WHERE membercost =0

/* Q3: Write an SQL query to show a list of facilities that charge a fee to members,
where the fee is less than 20% of the facility's monthly maintenance cost.
Return the facid, facility name, member cost, and monthly maintenance of the
facilities in question. */

SELECT facid, name, membercost, monthlymaintenance
FROM Facilities
WHERE (
membercost < monthlymaintenance * .2
)

/* Q4: Write an SQL query to retrieve the details of facilities with ID 1 and 5.
Try writing the query without using the OR operator. */

SELECT *
FROM Facilities
WHERE facid
IN ( 1, 5 )



/* Q5: Produce a list of facilities, with each labelled as
'cheap' or 'expensive', depending on if their monthly maintenance cost is
more than $100. Return the name and monthly maintenance of the facilities
in question. */

SELECT name, monthlymaintenance,
CASE WHEN monthlymaintenance > 100 THEN 'expensive'
WHEN monthlymaintenance = 100 THEN 'cheap'
ELSE 'cheap'
END AS Label
FROM Facilities




/* Q6: You'd like to get the first and last name of the last member(s)
who signed up. Try not to use the LIMIT clause for your solution. */

SELECT surname, firstname
FROM Members
WHERE joindate = (
SELECT MAX( joindate )
FROM Members )



/* Q7: Produce a list of all members who have used a tennis court.
Include in your output the name of the court, and the name of the member
formatted as a single column. Ensure no duplicate data, and order by
the member name. */

SELECT DISTINCT CONCAT(surname, ', ', firstname) AS name
FROM Members
INNER JOIN Bookings
ON Members.memid = Bookings.memid
WHERE Members.memid IN (
    SELECT DISTINCT memid 
	FROM Bookings
	WHERE bookid IN (
        SELECT bookid
		FROM Bookings
		WHERE facid IN (0,1) 
        )
    )
ORDER BY name
        

/* Q8: Produce a list of bookings on the day of 2012-09-14 which
will cost the member (or guest) more than $30. Remember that guests have
different costs to members (the listed costs are per half-hour 'slot'), and
the guest user's ID is always 0. Include in your output the name of the
facility, the name of the member formatted as a single column, and the cost.
Order by descending cost, and do not use any subqueries. */

SELECT
CONCAT(surname, ', ', firstname) AS name,
name AS facility,
CASE WHEN firstname = 'GUEST' THEN guestcost * slots ELSE membercost * slots END AS cost
FROM Members
INNER JOIN Bookings
ON Members.memid = Bookings.memid
INNER JOIN Facilities
ON Bookings.facid = Facilities.facid
WHERE starttime >= '2012-09-14' AND starttime < '2012-09-15'
AND CASE WHEN firstname = 'GUEST' THEN guestcost * slots ELSE membercost * slots END > 30
ORDER BY cost DESC


/* Q9: This time, produce the same result as in Q8, but using a subquery. */
SELECT * 
FROM (
SELECT
CONCAT(surname, ', ', firstname) AS member,
name AS facility,
CASE WHEN firstname = 'GUEST' THEN guestcost * slots ELSE membercost * slots END AS cost
FROM Members
INNER JOIN Bookings USING(memid)
INNER JOIN Facilities USING(facid)
WHERE starttime >= '2012-09-14' AND starttime < '2012-09-15') AS subquery
WHERE cost >30
ORDER BY cost DESC




/* PART 2: SQLite

Export the country club data from PHPMyAdmin, and connect to a local SQLite instance from Jupyter notebook 
for the following questions.  



QUESTIONS:
/* Q10: Produce a list of facilities with a total revenue less than 1000.
The output of facility name and total revenue, sorted by revenue. Remember
that there's a different cost for guests and members! */
   SELECT 
	f.name as facility, 
	SUM(b.slots * (CASE WHEN b.memid = 0 THEN f.guestcost ELSE f.membercost END)) AS revenue
FROM 
	Bookings b
INNER JOIN 
	Facilities f 
ON 
	b.facid = f.facid
GROUP BY 
	facility
HAVING 
	revenue < 1000
ORDER BY 
	revenue


/* Q11: Produce a report of members and who recommended them in alphabetic surname,firstname order */

SELECT Members.firstname AS Member_First, Members.surname AS Member_Last, Recommender.firstname AS Rec_first, Recommender.surname AS _Rec_last
FROM Members 
LEFT OUTER JOIN Members Recommender
ON Recommender.memid = Members.recommendedby
ORDER BY Members.surname, Members.firstname



/* Q12: Find the facilities with their usage by member, but not guests */

SELECT f.name, m.surname, m.firstname, COUNT(b.slots) AS Utilization
FROM Facilities AS f
JOIN Bookings b on f.facid = b.facid
JOIN Members m on b.memid = m.memid
WHERE m.surname <> 'guest'
GROUP BY f.name, m.surname, m.firstname



/* Q13: Find the facilities usage by month, but not guests */
SELECT f.name,
	CASE WHEN EXTRACT(MONTH FROM starttime) = 7 THEN 'Jul'
	WHEN EXTRACT(MONTH FROM starttime) = 8 THEN 'Aug'
	WHEN EXTRACT(MONTH FROM starttime) = 9 THEN 'Sep' END AS Month, 
	COUNT(b.slots) AS Utilization
FROM Facilities AS f
JOIN Bookings b on f.facid = b.facid
JOIN Members m on b.memid = m.memid
WHERE m.surname <> 'guest'
GROUP BY f.name, Month
