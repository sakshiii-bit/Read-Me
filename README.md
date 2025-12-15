1. Signed up for Snowflake. Created a free Snowflake trial account.
2. Signed up for Hevo via Snowflake Partner Connect.
3. Set up Postgres using Aiven.
Install a PostgreSQL database locally (Docker or VM) - I tried to install docker locally but it wasn't running then I tried ngrok but it was asking for card details so I wasn't able to use this as well. Then at last I got this aiven which was free so I used it.

4. Create tables & load data
Create the three tables (customers, orders, feedback) as defined in the schema and load
them with the provided CSVs -
- For this I have used Dbvisualizer to create tables and load the data. I connected it with my Aiven credentials.


5. Set up Hevo pipeline:
○ Source = PostgreSQL 
○ Destination = Snowflake
○ Ingestion mode must be Logical Replication

- Now I created this Hevo pipeline and given postgress Aiven credentials in it and once it was connected I added the destination as snowflake.

6. Apply Transformations:

- For applying the transformation, I have written this code for validation in the transformation tab in hevo:

from io.hevo.api import Event

def transform(event):
    events = []

    event_name = event.getEventName()
    properties = event.getProperties()

    if event_name == 'orders':
        status = properties.get('status')

        status_to_event = {
            'placed': 'order_placed',
            'shipped': 'order_shipped',
            'delivered': 'order_delivered',
            'cancelled': 'order_cancelled'
        }

        event_type = status_to_event.get(status)

        if event_type:
            order_event_props = {
                'order_id': properties.get('id'),
                'customer_id': properties.get('customer_id'),
                'event_type': event_type
            }

            events.append(Event('order_events', order_event_props))


    elif event_name == 'customers':
        email = properties.get('email')

        if email and '@' in email:
            username = email.split('@')[0]
            properties['username'] = username

        events.append(Event('customers', properties))

    else:
        events.append(Event(event_name, properties))

    return events

7. And lastly when I checked the data in GET SAMPLE and all results were coming right. I have deployed the code and saved the pipeline.

