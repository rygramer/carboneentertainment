# Carbone Entertainment Event Management Application

## A case study describing the implementation of a custom Salesforce application

[Carbone Entertainment](https://carboneentertainment.com/) is a talent agency that specializes in pairing you with the perfect artist, performer, and activity for your event. As a simple example, let’s say you are hosting your child’s birthday party and would like to hire the most amazing [Face Painter](https://carboneentertainment.com/service/face-painters/), Carbone connects you with that artist and handles the contracting, logistical coordination, and billing. Things get more complicated when the scale of the event increases to include many performers, a complicated venue, or an entire performance series across multiple days & months.

In order to streamline the business operation, I created a custom Salesforce application to manage the sales and fulfillment processes from end-to-end. The platform centralizes the talent database, contract management system, and customer engagement platform.

This case study will describe how I implemented the custom Salesforce application to improve our business by outlining the:

1. Data Structure;
2. Contracting Solution; and
3. Problems Solved by the Application.

## Data Structure

### Objects

As the Salesforce [documentation](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_concepts.htm) states, “... objects represent database tables that contain your organization's information.” Our application uses the Account and Contact standard objects, and relies heavily on the following custom objects:

*   **Event** - this object stores important information relevant to each event:
    *   Who is the client?
    *   What is the start and end date / time?
    *   What is the venue?
    *   What is the event type?
*   **Venue** - a venue is associated with each event:
    *   What is the venue name?
    *   What is the street address?
    *   What are the parking and load-in instructions specific to this event?
*   **Service** - these are the services that we (and our talent) provide for events:
    *   What is the service name?
    *   What is the link to the service on the Carbone website?
    *   What is the standard price for the service?
    *   What is included in the service?
    *   What is required by the client in order to perform the service?
*   **Talent** - as mentioned, we use the Account and Contact standard objects to store the data of each entity / person that we do business with. If these individuals are service providers that we pair with clients we create a record for them on the Talent object. In addition to linking to Accounts / Contacts, the Talent object also links to the Service object. We have created a database of providers based on their skill set and services they offer.
    *   What is the talent’s personal website?
    *   What is the talent’s standard price?
    *   Does the talent have any files we can relate to their record through the [Dropbox integration](https://appexchange.salesforce.com/appxListingDetail?listingId=a0N30000000prqLEAQ)?
*   **Event Service** - similar to the Products - Opportunities relationship, the Event Service object stores the data of the specific Services we are providing for a specific Event. As denoted in the name, the Event Service object connects the Event object to the Service object. These records appear as a line item on the client’s contract and invoice.
    *   Which service(s) are we providing for this event?
    *   Should the start and end date / time inherit the corresponding value from the Event object? If no, what is the start and end date / time for this service?
    *   Should the price inherit the corresponding value from the Service object? If no, what is the price of this service?
    *   What should the talent wear in terms of their wardrobe when performing this service for the client?
    *   What time should the talent arrive onsite for the event?
*   **Job Sheet** - this is Carbone Entertainment’s ‘fancy’ word for subcontract. For every event that is fulfilled by one of our amazing performers / artists (our talent), we provide them with a detailed job information sheet to spell out everything they could possibly need to know about the event. Job Sheets are assigned to each Event and are individually written for a specific Account / Contact.
    *   For which event is this job sheet?
    *   Who is the recipient of this job sheet?
    *   Are there any notes that we should include for the recipient regarding this event?
*   **Job Sheet Service** - Similar to the Event Services, the Job Sheet Service record(s) appear as line items on the talent’s subcontract. Job Sheet Services connect Job Sheets (which are connected to Events) and Event Services (which are connected to Services) and Talent (which are connected to Accounts / Contacts). Now we start to see a chain of connections with the goal being to decrease instances of double data entry. Enter data ONCE, and it is disseminated across the board. Need to customize data points at various steps? No problem; customization points have been built in.
    *   For which job sheet is this job sheet service?
    *   For which event service is this job sheet service?
    *   For which talent is this job sheet service?
    *   Should the price inherit the corresponding value from the Talent object? If no, what is the price of this job sheet service?
    *   Should the wardrobe inherit the corresponding value from the Event Service object? If no, what is the wardrobe?
    *   Should the arrival time inherit the corresponding value from the Event Service object? If no, what is the arrival time?

### Schema

For my visual readers as I know even I am a little dizzy from my object descriptions.

![Schema](/img/schema.png)

