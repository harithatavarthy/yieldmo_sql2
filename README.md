# yieldmo_sql2
There was a partial outage for an hour on 02/01 from 9:15 AM UTC to 10:15 UTC which led to a degraded user experience. It has been decided to refund $10000 to the users who were logged in during the partial outage. The refunds are to applied weighted based on seconds.  Using table below , write a query to get per user refund amount:  

# Solution

## Define the DDL/schema to hold session data

      CREATE TABLE IF NOT EXISTS login_event(
        session_id int NOT NULL,
        event_time timestamp,
        event_type varchar(200),
        user_id    int ,
        PRIMARY KEY (session_id, event_type)
      ) ;
      
## Insert values into the above defined schemas

      INSERT INTO login_event (session_id, event_time, event_type, user_id) VALUES
        ('100', '2019/02/01 9:00', 'login', 1),
        ('200', '2019/02/01 9:30', 'login', 2),
        ('100', '2019/02/01 10:00', 'logout', 1),
        ('300', '2019/02/01 10:00', 'login', 3),
        ('300', '2019/02/02 11:15', 'logout', 3),
        ('200', '2019/02/02 11:45', 'logout', 2),
        ('401', '2019/02/01 09:30', 'login', 4),
        ('401', '2019/02/01 10:00', 'logout', 4),
        ('501', '2019/02/01 09:00', 'login', 5),
        ('501', '2019/02/01 09:10', 'logout', 5)

       ;

        
# Now we will write and execute a sql script to identy users who had an active session during the outage and compute the total usage in second by all active users. 
 
      select  sum(wrap2.usage_in_secs) from
        (
        select wrap.user_id, EXTRACT(EPOCH FROM (wrap.login_time - wrap.last_event)) as usage_in_secs
         from ( 
          SELECT *,case when event_time > '2019/02/01 09:15' then least('2019/02/01 10:15',event_time)  else '2019/02/01 09:15' end as              login_time,
           greatest('2019/02/01 09:15', LAG(event_time,1) OVER
           (PARTITION BY session_id ORDER BY session_id, event_time, event_type)) AS last_event
           FROM login_event
           where 
           (event_type = 'login' and event_time < '2019/02/01 10:15' )
           or
           (event_type = 'logout' and event_time > '2019/02/01 09:15' )
        ) as wrap where wrap.event_type = 'logout' order by wrap.user_id
      ) as wrap2 ;
 
 # Output of the above SQL run
 
        sum
        ----
        8100
        
# Now we will write and re-execute the above sql but this time instead of find the sum of usage in seconds by all users, we will rather compute the refund by using the output of the above query as in input.
 
      select  wrap2.user_id, (10000*wrap2.usage_in_secs)/ 8100 as refund_$ from
        (
        select wrap.user_id, EXTRACT(EPOCH FROM (wrap.login_time - wrap.last_event)) as usage_in_secs
         from ( 
          SELECT *,case when event_time > '2019/02/01 09:15' then least('2019/02/01 10:15',event_time)  else '2019/02/01 09:15' end as              login_time,
           greatest('2019/02/01 09:15', LAG(event_time,1) OVER
           (PARTITION BY session_id ORDER BY session_id, event_time, event_type)) AS last_event
           FROM login_event
           where 
           (event_type = 'login' and event_time < '2019/02/01 10:15' )
           or
           (event_type = 'logout' and event_time > '2019/02/01 09:15' )
        ) as wrap where wrap.event_type = 'logout' order by wrap.user_id
      ) as wrap2 ;
 
 # Output of the above SQL run . The value under the column refund_$ will be refund amount to the user based on his/her usage during the outage. 
 
        user_id	refund_$
        ----------------------
        1	3333.3333333333335
        2	3333.3333333333335
        3	1111.111111111111
        4	2222.222222222222
   
   
   
  
   
 
  
