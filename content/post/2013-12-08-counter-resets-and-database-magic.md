---
title: Counter Resets and Database Magic
date: 2013-12-08
publishdate: 2013-12-08
---

Recently, the need for a standard numbering convention came up for one of our projects at work. The client 
requested to be able to access protocols by a unique ID. Originally I thought it'd be a no brainer, I'd explain 
to them that the each protocol is already uniquely identifiable within the database, and they can just refer to 
protocols by their unique ID in the database. Unfortunately, they didn't seem too fond of the unique ID numbering 
scheme and proposed that the application follow a certain convention, which they created.

The convention was simple enough to understand. It followed the format of:
    
    YYYY-XXX

Where `YYYY` was the Academic Year and `XXX` was a 3 digit number signifying that it was the nth protocol 
scheduled for review of that academic year. For example, a protocol with the number

    1314-016

Would be the 16th protocol scheduled within the 2013-2014 academic year.

While the standard is easy enough, it brings up some concerns.

1. Protocol numbers are generated once a protocol has been scheduled for committee review.
2. Protocol numbers must refer to a specific protocol.
3. The protocol number must reset after every academic year.
4. How can I enforce this standard within the database?

Regarding the first concern, I couldn't simply append a column to the protocol table since I would have a `NULL` 
field until the protocol was scheduled (if it was ever scheduled). While adding a column would allow me to specify
which number belonged to which protocol, it invalidate the 3NF standard I'm trying to achieve. Also, how 
would I have the database automatically increment or reset the value of the counter?

I realized quickly that I needed another table (or two). Immediately, I realized that my `protocol_number` 
table would need the following columns:

* ID - mainly for reference
* Protocol ID - specifies which number this protocol refers to
* Academic Year - the year part of the number convention
* Protocol Number - the counter part of the number convention

While designing the table, I also realized that I didn't want to have to worry about the application needing to 
worry about which protocol number to insert within the column and that lead me to the idea of a trigger. I know 
that I can easily count the number of rows within a table using the SQL command `count(*)`, but how could I use 
it for handling the protocol number field? I realized that my answer would rely in a trigger that would fire 
before each insert on the table.

Before jumping into creating the trigger, I had to give some more thought to the the academic year field. While 
I could easily represent academic years with an `INT` field, it didn't seem to be sufficient, something just felt 
dirty. I made the decision that I'd create a new table, `academic_year`, which would simply store academic years. 
This way, I could have the `protocol_number` table refer to the `academic_year` table as a foreign key.
Specifying the field as a foreign key also provided me the added benefit of being able to lookup which protocols
were scheduled for what year (a feature that hasn't been asked for, but can now be easily implemented).

Given this analysis, we've composed two tables with the following structure:
{{< highlight mysql >}}
CREATE TABLE academic_year(
	id int UNSIGNED NOT NULL AUTO_INCREMENT,
	academic_year int UNSIGNED NOT NULL,
	PRIMARY KEY(id)
);

CREATE TABLE protocol_number(
	id int UNSIGNED NOT NULL AUTO_INCREMENT,
	academic_year_id int UNSIGNED NOT NULL,
	protocol_id int UNSIGNED NOT NULL,
	protocol_number int UNSIGNED NOT NULL,
	FOREIGN KEY (protocol_id) REFERENCES protocols(id),
    KEY (id)
);
{{< / highlight >}}

Almost complete. We still have one tiny little problem. We want to ensure that our protocol numbers are unique 
and also that they're unique to a single protocol. Using primary keys we can specify that our `protocol_number` 
and `academic_year_id` columns are unique. In addition, we can easily add a unique key on our `protocol_id` field 
allowing us to ensure that this field is also unique. Doing so creates the following structure:

{{< highlight mysql >}}
CREATE TABLE protocol_number(
	id int UNSIGNED NOT NULL AUTO_INCREMENT,
	academic_year_id int UNSIGNED NOT NULL,
	protocol_id int UNSIGNED NOT NULL,
	protocol_number int UNSIGNED NOT NULL,
	FOREIGN KEY (protocol_id) REFERENCES protocols(id),
	UNIQUE KEY unique_protocol_id (protocol_id),
	FOREIGN KEY (academic_year_id) REFERENCES academic_year(id),
	PRIMARY KEY (protocol_number, academic_year_id),
	KEY (id)
);

{{< / highlight >}}

Finally, we can create our trigger to handle incrementing the counter. Our trigger will be based around the query

{{< highlight mysql >}}
SELECT COUNT(*) as num_protocols 
FROM eirb_number 
WHERE academic_year_id=[ACADEMIC_YEAR_ID]
{{< / highlight >}}

The query will return the number of rows that exist with the given `ACADEMIC_YEAR_ID`. All we would have to do 
now is take the results of this query, increment it by 1, and then set it as the value for the 
`protocol_number` column for the new row being inserted into the database. Given our analysis it sounds like 
we have all we need to create our a trigger, that is, when it occurs, `BEFORE INSERT` and  what we want to 
add/change `protocol_number` with our `count(*)` value.

Behold, our trigger:
{{< highlight mysql >}}
delimiter //
CREATE TRIGGER insert_protocol_num_trigger 
BEFORE INSERT on eirb_number
for each row
BEGIN
	SET new.protocol_number = (
		SELECT COUNT(*) as num_protocols 
		FROM eirb_number 
		WHERE academic_year_id=new.academic_year_id
	) + 1;
END; //
DELIMITER ;
{{< / highlight >}}

Now all our queries/application have to worry about is what protocol they want to create the number for and 
which academic year that protocol belongs to. The database will handle the rest. Our insert query can now look 
something like:

{{< highlight mysql >}}
INSERT INTO protocol_number(protocol_id, academic_year_id) VALUES(4, 1);
{{< / highlight >}}

Now we've handled the case of reset counters through the use of database magic (AKA triggers).

<!---
 Protocols can also have renewals and 
revisions, which would follow a similar format, just with certain extensions. If the protocol from the previous 
example was renewed for another year, then the renewed protocol would have the number:

    1314-016-a

The appended letters `a-z` would sig
-->
