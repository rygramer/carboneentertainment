# Carbone Entertainment Event Management Application


## A case study describing the implementation of a custom Salesforce application

[Carbone Entertainment](https://carboneentertainment.com/) is a talent agency that specializes in pairing you with the perfect artist, performer, and activity for your event. As a simple example, let’s say you are hosting your child’s birthday party and would like to hire the most amazing [Face Painter](https://carboneentertainment.com/service/face-painters/), Carbone connects you with that artist and handles the contracting, logistical coordination, and billing. Things get more complicated when the scale of the event increases to include many performers, a complicated venue, or an entire performance series across multiple days or months.

In order to streamline the business operation, I created a custom Salesforce application to manage the sales and fulfillment processes from end-to-end. The platform centralizes the talent database, contract management system, and customer engagement platform.

This case study will describe how I implemented the custom Salesforce application to improve our business by outlining the:



1. Data Structure;
2. Contracting Solution; and
3. Problems Solved by the Application.


## Data Structure


### Objects

The Salesforce [documentation](https://developer.salesforce.com/docs/atlas.en-us.object_reference.meta/object_reference/sforce_api_objects_concepts.htm) states, “... objects represent database tables that contain your organization's information.” In addition to using the Account and Contact standard objects, we have created the following custom objects:



*   **Event** - this object stores important information relevant to each Event.
    *   Who is the client?
    *   What is the start and end date / time?
    *   What is the venue?
    *   What is the event type?
*   **Venue** - a Venue is associated with each Event.
    *   What is the venue name?
    *   What is the street address?
    *   What are the parking and load-in instructions specific to this event?
*   **Service** - these are the Services that we (i.e. our Talent) provide for Events.
    *   What is the service name?
    *   What is the link to the service on the Carbone website?
    *   What is the standard price for the service?
    *   What is included in the service?
    *   What is required by the client in order to perform the service?
*   **Talent** - we use the Account and Contact standard objects to store the data of each entity and person that we do business with. If they are service providers, we create a record for them on the Talent object. In addition to linking to Accounts / Contacts, the Talent object also links to the Service object. This has allowed us to create a database of providers based on their skill set and services they offer.
    *   What is the talent’s personal website?
    *   What is the talent’s standard price?
    *   Does the talent have any files we can relate to their record through the [Dropbox integration](https://appexchange.salesforce.com/appxListingDetail?listingId=a0N30000000prqLEAQ)?
*   **Event Service** - similar to the Products - Opportunities relationship, the Event Service object stores the data of the specific Services we are providing for a specific Event. As denoted in the name, the Event Service object connects the Event object to the Service object. These records appear as a line item on the client’s contract and invoice.
    *   Which service(s) are we providing for this event?
    *   Should the start and end date / time inherit the corresponding value from the Event object? If no, what is the start and end date / time for this service?
    *   Should the price inherit the corresponding value from the Service object? If no, what is the price of this service?
    *   What should the talent wear in terms of their wardrobe when performing this service for the client?
    *   What time should the talent arrive onsite for the event?
*   **Job Sheet** - this is Carbone’s word for subcontract. For every Event that is fulfilled by our Talent, we provide them with a detailed job information sheet to communicate the logistical details of the Event. Job Sheets are assigned to each Event and are individually written for a specific Account / Contact.
    *   For which event is this job sheet?
    *   Who is the recipient of this job sheet?
    *   Are there any notes that we should include for the recipient regarding this event?
*   **Job Sheet Service** - Similar to Event Services, the Job Sheet Service records appear as line items on the Talent’s subcontract. Job Sheet Services connect Job Sheets (which are connected to Events), Event Services (which are connected to Services), and Talent (which are connected to Accounts / Contacts). Now we start to see a chain of connections with the goal being to decrease instances of double data entry. We enter data ONCE, and it is disseminated across the board. Need to customize at any step in the process? Customization points have been built in.
    *   For which job sheet is this job sheet service?
    *   For which event service is this job sheet service?
    *   For which talent is this job sheet service?
    *   Should the price inherit the corresponding value from the Talent object? If no, what is the price of this job sheet service?
    *   Should the wardrobe inherit the corresponding value from the Event Service object? If no, what is the wardrobe?
    *   Should the arrival time inherit the corresponding value from the Event Service object? If no, what is the arrival time?
*   **Other Objects** -
    *   **Signed Contract** - when a client is sent a contract (from the Event object), a personal link is provided for them to digital sign the contract via the Carbone website. The [Gravity Forms Salesforce Add-on](https://wordpress.org/plugins/gravity-forms-salesforce/) allows the system to automatically create a record on this object every time a contract is signed.
    *   **Payment** - any time a Payment is made towards an Event, a record is created. If the client pays with a credit card or echeck via the Carbone website, the record is created automatically.
    *   **Expense** - if we incur an additional expense related to an event (aside from payroll liabilities represented on Job Sheets), we create a record to track the cost, receipt, and memo. This allows us to better track gross profit per Event.
    *   **Check-in** - Similar to the Signed Contract object to track client signatures, the Check-in object tracks signed Job Sheets. The Check-in object comes in two varieties - before the event responses and after the event responses. In addition to receiving signatures before the event, we provide Talent the opportunity to ask any questions they may have or let us know how the event went. Reports are generated to track their responses.


### Schema

![Schema](/img/schema.png)


## Contracting Solution

A critical feature required of the application is the ability to generate, send, and track contracts. We work with two different contractual documents - a contract for clients and a subcontract for talent. To support this solution, I have written a series of Visualforce Pages, Visualforce Email Templates, Apex Classes, and Unit Tests.


### Emailing the Contract

After the contract has been prepared and internally reviewed, it is ready to be sent via email. When the custom checkbox field is selected, a custom formula field populates with a link to “Send Contract”.

![Select the ‘ready for contract” checkbox in order to send the contract](/img/ready-for-contract.png)

The link navigates to a custom Visualforce Page that allows the user to review the contract. The contract is a custom Visualforce Page rendered as a PDF.


```
<apex:page standardController="Event__c" title="{!Event__c.Name} : Preview Contract" showHeader="false" sidebar="false" standardStylesheets="false" renderAs="PDF" applyHtmlTag="false" applyBodyTag="false" lightningstylesheets="false">
<html>
    <head>
    <title>Service Agreement for {!Event__c.Name}</title>
        <style type="text/css" media="print,screen">
            @page {
                margin: 1in .5in;
                @top-left {
                    content: element(emoji);
                };
                @top-center {
                    padding-top:1em;
                    vertical-align:top;
                    content: element(logo);
                };
                @top-right {
                    padding-top: 1em;
                    font-family:sans-serif;
                    font-size:.8em;
                    content: "{!Event__c.Number__c}";
                };
                @bottom-left {
                    content: element(now);
                }
                @bottom-center {
                    content: element(contact);
                };
                @bottom-right {
                    content: element(page-number);
                };                                
             }
             body {
                font-family:sans-serif;
             }
             .event-name {
                font-size:1.75em;
                padding-bottom:.25em;                
             }
             .date {
                font-size:1.25em;
                padding-bottom:1em;
             }             
             #emoji {
                 vertical-align:middle;
                 position: running(emoji);
             }
             #logo{
                position: running(logo);
             }
             #now{
                 font-family:sans-serif;
                 font-size:.8em;
                 text-align:center;
                 position: running(now);
             }
             #contact{
                 font-family:sans-serif;
                 font-size:.8em;
                 text-align:center;
                 position: running(contact);
             }
             #page-number {
                font-family:sans-serif;
                font-size:.8em;
                text-align:right;
                position: running(page-number);
             }
             .client, .venue {
                 text-align:center;
             }
            .pagenumber:before {
                content: counter(page);
            }
            .pagecount:before {
                content: counter(pages);
            }
            .service-columns {
                vertical-align:top;
            }
            .overtime {
                color:red;
            }
            p.indent {
                text-indent:1em;
            }
            .underline {
                text-decoration: underline;
            }
            li {
                margin-bottom:1em;
            }
            .ordered-list{
                padding-left:1em;
            }
            .number{
                display:block;
                text-indent:1em;
                margin-bottom:1em;
            }
            ol.sub{
                list-style-type: lower-alpha;
                margin-left:4em;
            }
            a{
                color: #6677EE ; font-family: Helvetica, Arial, sans-serif; font-weight: normal; line-height: 1.3; margin: 0; padding: 0; text-align: left; text-decoration: underline;
            }
        </style>
    </head>
    <body>
        <div id="emoji">
            <img src="{!Event__c.Event_Type_Emoji__c}" height="48" />
        </div>       
        <div id="logo">
            <center><apex:image value="{!$Resource.ce_logo_101917}" height="48"/></center>
        </div>
        <div id="page-number">
            <p>Page <span class="pagenumber"/> of <span class="pagecount"/></p>
        </div>
        <div id="contact">
            Carbone Entertainment, Inc.<br />
            12346 Sandy Point Court, Silver Spring, MD 20904<br />
            office: (301) 572-7717 | fax: (301) 572-7716<br />
            <span style="color:red;">emergency: (410) 251-0650</span><br />
        </div>
        <div id="now">
            <apex:outputText value=" {!NOW()}" /><apex:outputText value=" UTC" rendered="{!IF($User.Alias="guest",true,false)}"/>
        </div>
        <center>Summary Sheet</center>
        <center><div class="event-name">{!Event__c.Name}</div></center>
        <center>
            <div class="date">
                <apex:outputText rendered="{!IF($User.Alias='guest',false,true)}">
                      <apex:outputText value="{0, date, EEEE',' MMMM d','  yyyy}">
                          <apex:param value="{!Event__c.Start_Time__c-Event__c.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                      <apex:outputText value=" - {0, date, EEEE',' MMMM d','  yyyy}" rendered="{!IF(DATEVALUE(Event__c.Start_Time__c)!=DATEVALUE(Event__c.End_Time__c),true,false)}">
                          <apex:param value="{!Event__c.End_Time__c-Event__c.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                </apex:outputText>
                <apex:outputText rendered="{!IF($User.Alias='guest',true,false)}">
                      <apex:outputText value="{0, date, EEEE',' MMMM d','  yyyy}">
                          <apex:param value="{!Event__c.Start_Time__c-Event__c.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                      <apex:outputText value=" - {0, date, EEEE',' MMMM d','  yyyy}" rendered="{!IF(DATEVALUE(Event__c.Start_Time__c-Event__c.Daylight_Savings_Calculation__c)!=DATEVALUE(Event__c.End_Time__c-Event__c.Daylight_Savings_Calculation__c),true,false)}">
                          <apex:param value="{!Event__c.End_Time__c-Event__c.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                </apex:outputText>
            </div>
        </center>       
        
        <apex:pageBlock >
           <apex:pageblockTable value="{!Event__c}" var="a" align="center" columnswidth="360px,360px" style="padding-bottom:1em">
               
               <apex:column headerValue="Client" style="text-align:center;vertical-align:top;font-size:.9em;" headerClass="client">
                    <apex:outputText rendered="{!IF(Event__c.Account__r.RecordType.Name='Person Account',false,true)}">
                        {!Event__c.Account__r.Name}<br />
                    </apex:outputText>
                     {!Event__c.Client__r.Name}<br />
                    <apex:outputText rendered="{!IF(ISBLANK(Event__c.Account__r.BillingStreet),false,true)}">
                        {!Event__c.Account__r.BillingStreet}<br />
                        {!Event__c.Account__r.BillingCity}, {!Event__c.Account__r.BillingState} {!Event__c.Account__r.BillingPostalCode}<br />
                    </apex:outputText>
                    <apex:outputText rendered="{!IF(ISBLANK(Event__c.Client__r.MobilePhone),false,true)}">
                        <span style="font-style:italic;">m</span> {!Event__c.Client__r.MobilePhone}<br />
                    </apex:outputText>
                    <apex:outputText rendered="{!IF(ISBLANK(Event__c.Client__r.Phone),false,true)}">                    
                        <span style="font-style:italic;">o</span> {!Event__c.Client__r.Phone}<br />
                    </apex:outputText>
                    <apex:outputText rendered="{!IF(ISBLANK(Event__c.Client__r.Fax),false,true)}">                    
                        <span style="font-style:italic;">f</span> {!Event__c.Client__r.Fax}<br />
                    </apex:outputText>
                    <apex:outputText rendered="{!IF(ISBLANK(Event__c.Client__r.Email),false,true)}">                                                          
                         {!Event__c.Client__r.Email}<br />
                    </apex:outputText>
                    
                    <apex:outputText rendered="{!IF(ISBLANK(Event__c.c_o__c),false,true)}">
                        <br/>
                        c/o {!Event__c.c_o__r.Name}<br/>
                        <apex:outputText rendered="{!IF(ISBLANK(Event__c.c_o__r.MobilePhone),false,true)}">
                            <span style="font-style:italic;">m</span> {!Event__c.c_o__r.MobilePhone}<br />
                        </apex:outputText>
                        <apex:outputText rendered="{!IF(ISBLANK(Event__c.c_o__r.Phone),false,true)}">                    
                            <span style="font-style:italic;">o</span> {!Event__c.c_o__r.Phone}<br />
                        </apex:outputText>
                        <apex:outputText rendered="{!IF(ISBLANK(Event__c.c_o__r.Fax),false,true)}">                    
                            <span style="font-style:italic;">f</span> {!Event__c.c_o__r.Fax}<br />
                        </apex:outputText>
                        <apex:outputText rendered="{!IF(ISBLANK(Event__c.c_o__r.Email),false,true)}">                                                          
                             {!Event__c.c_o__r.Email}<br />
                        </apex:outputText>
                   </apex:outputText>         
               </apex:column>
               
               <apex:column headerValue="Venue" style="text-align:center;vertical-align:top;font-size:.9em;" headerClass="venue">
                   {!Event__c.Venue__r.Name}<br />
                   {!Event__c.Venue__r.Venue_Street__c}<br />
                   {!Event__c.Venue__r.Venue_City__c}, {!Event__c.Venue__r.Venue_State__c} {!Event__c.Venue__r.Venue_Zip__c}<br />
                   <apex:outputText rendered="{!IF(NOT(ISBLANK(Event__c.Venue__r.Venue_Phone__c)),TRUE,FALSE)}">
                       {!Event__c.Venue__r.Venue_Phone__c}<br />
                   </apex:outputText>
                   <apex:outputText rendered="{!IF(NOT(ISBLANK(Event__c.Location_in_Venue__c)),TRUE,FALSE)}">
                       <div style="font-style:italic;">{!Event__c.Location_in_Venue__c}</div>
                   </apex:outputText>                                
               </apex:column>
           </apex:pageblockTable>
        </apex:pageBlock>

        <apex:pageBlock >
           <apex:pageblockTable value="{!Event__c.Event_Services__r}" var="b" cellpadding="5" style="font-size:.8em;border-top:1px dotted #D3D3D3;border-bottom:1px dotted #D3D3D3;padding-bottom:1em;" width="100%">
              
              <apex:column styleClass="service-columns" width="20%" style="text-align:center;">
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Service__r.Emoji_Link__c)||b.Event__r.Remove_Emoji__c=TRUE, false, true)}">
                      <img src="{!b.Service__r.Emoji_Link__c}" align="left" height="32px"/>
                  </apex:outputPanel>
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Service__r.Link_to_Website__c), false, true)}">
                      <a href="{!b.Service__r.Link_to_Website__c}"><b>{!b.Service__r.Name}<br /></b></a>
                  </apex:outputPanel>
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Service__r.Link_to_Website__c), true, false)}">
                      <b>{!b.Service__r.Name}<br /></b>
                  </apex:outputPanel>                                
              </apex:column>
              
              <apex:column width="45%" styleClass="service-columns">             
                 
                 <apex:outputText rendered="{!IF($User.Alias='guest',false,true)}">
                      <apex:outputText value="{0, time, h:mm a}" rendered="{!IF(ISBLANK(b.Actual_Start_Time__c), false, true)}">
                          <b>Start time:</b> <apex:param value="{!b.Actual_Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                      
                      <apex:outputText value="{0, date, 'on' EEE',' MMMM d','  yyyy}" rendered="{!IF(ISBLANK(b.Actual_Start_Time__c) || DATEVALUE(b.Event__r.Start_Time__c) = DATEVALUE(b.Event__r.End_Time__c) && DATEVALUE( b.Actual_Start_Time__c ) = DATEVALUE( b.Actual_End_Time__c ), false, true)}">
                          <apex:param value="{!b.Actual_Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText> 
                                       
                      <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_Start_Time__c), false, true)}"><br /></apex:outputPanel>
                      
                      <apex:outputText value="{0, time, h:mm a}" rendered="{!IF(ISBLANK(b.Actual_End_Time__c), false, true)}">
                          <b>End time:</b> <apex:param value="{!b.Actual_End_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                      <apex:outputText value="{0, date, 'on' EEE',' MMMM d','  yyyy}" rendered="{!IF(ISBLANK(b.Actual_Start_Time__c) || DATEVALUE(b.Event__r.Start_Time__c) = DATEVALUE(b.Event__r.End_Time__c) && DATEVALUE( b.Actual_Start_Time__c ) = DATEVALUE( b.Actual_End_Time__c ), false, true)}">
                          <apex:param value="{!b.Actual_End_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                  </apex:outputText>
                 
                 <apex:outputText rendered="{!IF($User.Alias='guest',true,false)}">
                      <apex:outputText value="{0, time, h:mm a}" rendered="{!IF(ISBLANK(b.Actual_Start_Time__c), false, true)}">
                          <b>Start time:</b> <apex:param value="{!b.Actual_Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                      
                      <apex:outputText value="{0, date, 'on' EEE',' MMMM d','  yyyy}" rendered="{!IF(ISBLANK(b.Actual_Start_Time__c) || DATEVALUE(b.Event__r.Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c) = DATEVALUE(b.Event__r.End_Time__c - b.Event__r.Daylight_Savings_Calculation__c) && DATEVALUE( b.Actual_Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c ) = DATEVALUE( b.Actual_End_Time__c - b.Event__r.Daylight_Savings_Calculation__c ), false, true)}">
                          <apex:param value="{!b.Actual_Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText> 
                                       
                      <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_Start_Time__c), false, true)}"><br /></apex:outputPanel>
                      
                      <apex:outputText value="{0, time, h:mm a}" rendered="{!IF(ISBLANK(b.Actual_End_Time__c), false, true)}">
                          <b>End time:</b> <apex:param value="{!b.Actual_End_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                      <apex:outputText value="{0, date, 'on' EEE',' MMMM d','  yyyy}" rendered="{!IF(ISBLANK(b.Actual_Start_Time__c) || DATEVALUE(b.Event__r.Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c) = DATEVALUE(b.Event__r.End_Time__c - b.Event__r.Daylight_Savings_Calculation__c) && DATEVALUE( b.Actual_Start_Time__c - b.Event__r.Daylight_Savings_Calculation__c ) = DATEVALUE( b.Actual_End_Time__c - b.Event__r.Daylight_Savings_Calculation__c ), false, true)}">
                          <apex:param value="{!b.Actual_End_Time__c - b.Event__r.Daylight_Savings_Calculation__c}" />
                      </apex:outputText>
                  </apex:outputText>
                  
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_End_Time__c), false, true)}"><br /></apex:outputPanel>                  
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_Inclusions__c), false, true)}"><b>What's included:</b> <apex:outputText value=" {!b.Actual_Inclusions__c}" escape="false" /><br /></apex:outputPanel>
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_Requirements__c), false, true)}"><b>What's needed:</b> <apex:outputText value=" {!b.Actual_Requirements__c}" escape="false" /><br /></apex:outputPanel>
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_Spatial_Requirements__c), false, true)}"><b>Spatial Requirements:</b> {!b.Actual_Spatial_Requirements__c}<br /></apex:outputPanel> 
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_Price_Description__c), false, true)}"><b>Price:</b> {!b.Actual_Price_Description__c}<br /></apex:outputPanel>
                  <apex:outputPanel rendered="{!IF(ISBLANK(b.Actual_OT_Price_Unit__c), false, true)}"><b>Overtime:</b> Is billed in half hour increments at a rate of ${!b.Actual_OT_Price_Unit__c}0/{!IF(b.Use_Custom_OT_Unit__c=TRUE,LOWER(b.Actual_OT_Unit__c),LOWER(b.Actual_Unit__c))}.<br /></apex:outputPanel>
                  <apex:outputPanel rendered="{!IF(b.Was_there_OT__c = TRUE, true, false)}" styleClass="overtime"><b>Overtime has been calculated in red.</b></apex:outputPanel>
              </apex:column>
              
              <apex:column styleClass="service-columns" style="text-align:center;" width="10%">
                  {!b.Actual_Quantity__c} {!LOWER(b.Actual_Unit__c)}{!IF(b.Actual_Quantity__c = 1, NULL, "s")}
                  <apex:outputPanel rendered="{!IF(b.Was_there_OT__c = TRUE, true, false)}" styleClass="overtime"><br /><b>{!b.OT_Quantity__c} {!IF(b.Use_Custom_OT_Unit__c=TRUE,LOWER(b.Actual_OT_Unit__c),LOWER(b.Actual_Unit__c))}{!IF(b.OT_Quantity__c = 1, NULL, "s")}</b></apex:outputPanel>
              </apex:column>
              
              <apex:column styleClass="service-columns" style="text-align:center;" width="15%">
                  <apex:outputText value="${0,number,###,###,###,##0.00}">
                      <apex:param value="{!b.Actual_Price_Unit__c}" />
                  </apex:outputText>/{!LOWER(b.Actual_Unit__c)}<br />
                  <apex:outputPanel rendered="{!IF(b.Was_there_OT__c = TRUE, true, false)}" styleClass="overtime">
                      <b><apex:outputText value="${0,number,###,###,###,##0.00}"><apex:param value="{!b.Actual_OT_Price_Unit__c}" /></apex:outputText>/{!IF(b.Use_Custom_OT_Unit__c=TRUE,LOWER(b.Actual_OT_Unit__c),LOWER(b.Actual_Unit__c))}</b>
                  </apex:outputPanel>
              </apex:column>
                         
              <apex:column styleClass="service-columns" style="text-align:right;" width="10%">
                  <b><apex:outputText value="${0,number,###,###,###,##0.00}">
                      <apex:param value="{!b.Total_Price__c}" />
                  </apex:outputText></b><br />
                  <apex:outputPanel rendered="{!IF(b.Was_there_OT__c = TRUE, true, false)}" styleClass="overtime">
                      <b><apex:outputText value="${0,number,###,###,###,##0.00}"><apex:param value="{!b.Total_OT_Price__c}" /></apex:outputText></b>
                  </apex:outputPanel>
               </apex:column>
               
           </apex:pageblockTable>
        </apex:pageBlock>

        <apex:outputText>
            <div style="page-break-inside:avoid;">
                <table width="100%" style="page-break-inside:;" cellpadding="5px">
                    <tr>
                        <td width="85%">
                        <div style="padding-top:1em;float:right;">
                               <div style="text-align:center;font-size:1.1em;padding-bottom:.25em;">
                                   The signed contract
                                   <apex:outputPanel rendered="{!IF(Event__c.Are_we_expecting_a_deposit__c=TRUE, true, false)}">
                                        and <apex:outputText value=" {!VALUE(TEXT(Event__c.Deposit_Percentage__c))}%"/> deposit (<apex:outputText value="${0,number,###,###,###,##0.00}"><apex:param value="{!Event__c.Deposit_Amount__c}" /></apex:outputText>) are
                                   </apex:outputPanel> 
                                   <apex:outputPanel rendered="{!IF(Event__c.Are_we_expecting_a_deposit__c=TRUE, false, true)}">
                                       is
                                   </apex:outputPanel>                                
                                   due by<br />
                                   <b><apex:outputText value=" {0, date, EEEE',' MMMM d','  yyyy}">
                                           <apex:param value="{!IF( NOT( ISBLANK( Event__c.Signature_Deposit_Due_Date__c ) ) && $User = "guest" , Event__c.Signature_Deposit_Due_Date__c - Event__c.Daylight_Savings_Calculation__c , IF( Event__c.Use_Custom_Signature_Deposit_Due_Date__c=TRUE, Event__c.Signature_Deposit_Due_Date__c - Event__c.Daylight_Savings_Calculation__c ,
                                                IF(
                                                ( DATEVALUE(Event__c.Start_Time__c) - Today() ) > 14 , TODAY() + 7 ,
                                                IF(
                                                AND( ( DATEVALUE(Event__c.Start_Time__c) - Today() ) > 7 , ( DATEVALUE(Event__c.Start_Time__c) - TODAY() ) <= 14 ) , TODAY() + 3 ,
                                                IF(
                                                AND( ( DATEVALUE(Event__c.Start_Time__c) - Today() ) > 1 , ( DATEVALUE(Event__c.Start_Time__c) - TODAY() ) <= 7 ) , TODAY() + 1 ,
                                                IF(
                                                ( DATEVALUE(Event__c.Start_Time__c) - Today() ) = 1 , TODAY(), NULL
                                                ))))))}"/>
                                       </apex:outputText>.
                                   </b>                           
                               <br />                           
                               </div>
                               <div style="text-align:center;font-size:1.5em;">                        
                                   <apex:outputPanel rendered="{!IF(Event__c.Are_we_expecting_a_deposit__c=TRUE, true, false)}" style="margin-top:1em;">                    
                                       <a href="{!Event__c.Sign_Contract_Link__c}" style="text-decoration:none;">
                                           <apex:image value="{!$Resource.Point_Right2}" height="32px"/>
                                           <span style="vertical-align:15px;padding:0px 7px 0 5px;text-decoration:underline;">Sign contract &amp; Pay deposit</span>
                                           <apex:image value="{!$Resource.Point_Left2}" height="32px"/>
                                       </a>
                                   </apex:outputPanel>
                                   <apex:outputPanel rendered="{!IF(Event__c.Are_we_expecting_a_deposit__c=FALSE, true, false)}" style="padding-top:1em;">                    
                                       <a href="{!Event__c.Sign_Contract_Link__c}" style="text-decoration:none;">
                                           <apex:image value="{!$Resource.Point_Right2}" height="32px"/>
                                           <span style="vertical-align:20px;padding:0px 7px 0 5px;text-decoration:underline;">Sign Contract</span>
                                           <apex:image value="{!$Resource.Point_Left2}" height="32px"/>
                                       </a>
                                   </apex:outputPanel>     
                               </div>                      
                               <div style="text-align:center;font-size:1em;">    
                                   The final balance is due by<br />
                                   <b><apex:outputText value="{0, date, EEEE',' MMMM d','  yyyy}"><apex:param value="{!Event__c.Final_Payment_Due_Date__c-Event__c.Daylight_Savings_Calculation__c}"/></apex:outputText>.</b>
                               </div>
                               <br />
                               <div style="text-align:center;font-size:.8em;">    
                                   Parking must be reimbursed and/or provided by the client for all talent.
                               </div>
                               
                                <apex:outputText rendered="{!IF(ISBLANK(Event__c.Contract_Note__c),false,true)}" style="">        
                                   <div style="margin-top:2em;padding:1em;width:100%;background-color: #E1FAEA; border: 1px solid #22cd60;">                                         
                                       <apex:outputText value="{!Event__c.Contract_Note__c}" escape="false" />                      
                                   </div>       
                                </apex:outputText>                       
                                                                  
                           </div>             
                       </td>
                       
                       <td width="20%" valign="top">
                       <div style="float:right;padding-top:1em;">        
                           <apex:outputText rendered="{!IF(Event__c.Subtotal__c = Event__c.Final_Total__c, false, true)}">Subtotal<br /></apex:outputText>
                           <apex:outputText rendered="{!IF(ISBLANK(Event__c.Actual_Discount_Amount__c), false, true)}">Discount<br /></apex:outputText>
                           <apex:outputText rendered="{!IF(ISBLANK(Event__c.Actual_Parking_Amount__c), false, true)}">Parking<br /></apex:outputText>
                           <apex:outputText rendered="{!IF(ISBLANK(Event__c.Actual_Travel_Amount__c), false, true)}">Travel<br /></apex:outputText>
                           <apex:outputText rendered="{!IF(ISBLANK(Event__c.CC_Processing_Fee_Total__c)||Event__c.CC_Processing_Fee_Total__c=0, false, true)}">CC Fee<br /></apex:outputText>
                           <apex:outputText rendered="{!IF(Event__c.Subtotal__c = Event__c.Final_Total__c, false, true)}"><br /></apex:outputText>
                           <b>Total</b><br />
                           <br />
                           <apex:outputText rendered="{!IF(OR(ISBLANK(Event__c.Total_Payments__c),Event__c.Total_Payments__c=0), false, true)}">Payments<br /></apex:outputText>
                           <apex:outputText rendered="{!IF(OR(ISBLANK(Event__c.Total_Payments__c),Event__c.Total_Payments__c=0), false, true)}"><br />Balance Due<br /></apex:outputText>            
                       </div>
                       </td>
                       
                       <td width="10%" valign="top">
                       <div style="text-align:right;padding-top:1em;">        
                           <apex:outputPanel rendered="{!IF(Event__c.Subtotal__c = Event__c.Final_Total__c, false, true)}">
                               <apex:outputText value="${0,number,###,###,###,##0.00}">
                                   <apex:param value="{!Event__c.Subtotal__c}" />
                               </apex:outputText><br />
                           </apex:outputPanel>
                           <apex:outputPanel rendered="{!IF(ISBLANK(Event__c.Actual_Discount_Amount__c), false, true)}">
                               <apex:outputText value="(${0,number,###,###,###,##0.00})">
                                   <apex:param value="{!Event__c.Actual_Discount_Amount__c}" />
                               </apex:outputText><br />
                           </apex:outputPanel>                   
                           <apex:outputPanel rendered="{!IF(ISBLANK(Event__c.Actual_Parking_Amount__c), false, true)}">
                               <apex:outputText rendered="{!IF(ISBLANK(Event__c.Actual_Parking_Amount__c), false, true)}" value="${0,number,###,###,###,##0.00}">
                                   <apex:param value="{!Event__c.Actual_Parking_Amount__c}" />
                               </apex:outputText><br />
                           </apex:outputPanel>
                           <apex:outputPanel rendered="{!IF(ISBLANK(Event__c.Actual_Travel_Amount__c), false, true)}">
                               <apex:outputText rendered="{!IF(ISBLANK(Event__c.Actual_Travel_Amount__c), false, true)}" value="${0,number,###,###,###,##0.00}">
                                   <apex:param value="{!Event__c.Actual_Travel_Amount__c}" />
                               </apex:outputText><br />
                           </apex:outputPanel>
                           <apex:outputPanel rendered="{!IF(ISBLANK(Event__c.CC_Processing_Fee_Total__c)||Event__c.CC_Processing_Fee_Total__c=0, false, true)}">
                               <apex:outputText rendered="{!IF(ISBLANK(Event__c.CC_Processing_Fee_Total__c)||Event__c.CC_Processing_Fee_Total__c=0, false, true)}" value="${0,number,###,###,###,##0.00}">
                                   <apex:param value="{!Event__c.CC_Processing_Fee_Total__c}" />
                               </apex:outputText><br />
                           </apex:outputPanel>
                           <apex:outputText rendered="{!IF(Event__c.Subtotal__c = Event__c.Final_Total__c, false, true)}"><br /></apex:outputText>
                           <b><apex:outputText value="${0,number,###,###,###,##0.00}">
                               <apex:param value="{!Event__c.Final_Total__c}" />
                           </apex:outputText></b><br />
                           <br />
                           <apex:outputPanel rendered="{!IF(OR(ISBLANK(Event__c.Total_Payments__c),Event__c.Total_Payments__c=0), false, true)}">
                               <apex:outputText value="(${0,number,###,###,###,##0.00})">
                                   <apex:param value="{!Event__c.Total_Payments__c}" />
                               </apex:outputText><br />
                           </apex:outputPanel>
                           <apex:outputPanel rendered="{!IF(OR(ISBLANK(Event__c.Total_Payments__c),Event__c.Total_Payments__c=0), false, true)}">
                               <br /><apex:outputText value="${0,number,###,###,###,##0.00}">
                                   <apex:param value="{!Event__c.Balance_Due__c}" />
                               </apex:outputText><br />
                           </apex:outputPanel>              
                       </div>
                       </td>
                   </tr>                  
                </table>          
            </div>
        </apex:outputText>

        <apex:pageBlock >
           <apex:pageblockTable value="{!Event__c}" var="c" style="page-break-before:always;" width="100%" columns="1">
               <apex:column width="100%">    
                    <center>Services Agreement</center>
                        <apex:outputPanel rendered="{!IF(Event__c.Jan_Davis__c = true, false, true)}">
                            <div style="text-align:justify;font-size:.8em;">       
                                <p class="indent">THIS <b>SERVICES AGREEMENT</b> (the "Agreement") is made and entered into this
                                <apex:outputPanel >
                                    <apex:outputText value=" {0, date, MMMM d','  yyyy} "><apex:param value="{!TODAY()}" /></apex:outputText>
                                </apex:outputPanel>                
                                by and between Carbone Entertainment, Inc. ("Carbone"), a Maryland corporation having its principal place of business at 12346 Sandy Point Court, Silver Spring, MD 20904, and {!Event__c.Account__r.Name} (the "Customer"), having a principal address of {!Event__c.Account__r.BillingStreet}, {!Event__c.Account__r.BillingCity}, {!Event__c.Account__r.BillingState} {!Event__c.Account__r.BillingPostalCode} (with the parties collectively referred to herein as the "Parties").</p>
                                <p class="indent">WHEREAS, Carbone is an event entertainment agency that specializes in providing customers with artists, performers, and activities.</p> 
                                <p class="indent">WHEREAS, Customer intends to hold the event described on the "Summary Sheet" attached to this Agreement (the "Event");</p>
                                <p class="indent">WHEREAS, with regard to the Event, Customer desires to engage Carbone to perform the services described on the Summary Sheet (the "Services") and to do so pursuant to the terms set forth on the Summary Sheet;</p>
                                <p class="indent">WHEREAS, Carbone desires to provide Customer with the Services on the terms set forth on the Summary Sheet; and</p>
                                <p class="indent">WHEREAS, the Parties agree to such engagement, pursuant to the terms and conditions contained herein.</p>
                                <p class="indent">NOW, THEREFORE, the Parties hereby agree as follows:</p>
                                
                                    <span class="number">1.<span class="ordered-list"><span class="underline">Services.</span> Carbone shall perform the Services on the date and in accordance with the time frames and stipulations set forth on the Summary Sheet, which is fully incorporated by reference herein.</span></span>
                                    <span class="number">2.<span class="ordered-list"><span class="underline">Cooperation.</span> The Parties agree to cooperate in good faith to achieve satisfactory completion of the Services in a timely and professional manner.</span></span>
                                    <span class="number">3.<span class="ordered-list"><span class="underline">Payments.</span></span></span>
                                        <ol class="sub">
                                            <apex:outputText rendered="{!IF(Event__c.Are_we_expecting_a_deposit__c=TRUE,true,false)}">
                                                <li><span class="underline">Deposit.</span> Within seven (7) days of the parties' signing of this Customer shall provide Carbone with a <apex:outputText rendered="{!IF(Event__c.CreatedDate>=DATETIMEVALUE('2020-3-28 00:00:00'),true,false)}"> non-refundable</apex:outputText> deposit in the amount set forth on the Summary Sheet. That amount will be applied to the total non-"Overtime" price listed on the Summary Sheet (the "Compensation"), plus any additional charges, as set forth below.</li>
                                            </apex:outputText>
                                            <li><span class="underline">Cancellation.</span> If Customer cancels the Event, a cancellation fee will be immediately due and payable to Carbone. Cancellation fees shall be in the following amounts:  (i) fifty percent (50.0%) of the Compensation if cancellation occurred <apex:outputText rendered="{!IF(Event__c.CreatedDate>=DATETIMEVALUE('2020-3-28 00:00:00'),true,false)}"> eight (8) days or more</apex:outputText><apex:outputText rendered="{!IF(Event__c.CreatedDate>=DATETIMEVALUE('2020-3-28 00:00:00'),false,true)}"> eight (8) to fourteen (14) days</apex:outputText> prior to the scheduled Event date, (ii) seventy-five percent (75.0%) of the Compensation if cancellation occurred four (4) to seven (7) days prior to the scheduled Event date, and (iii) one hundred percent (100.0%) of the Compensation if cancellation occurred within three (3) days of the scheduled Event date.</li>
                                            <li><span class="underline">Rescheduling.</span> Unless (i) Customer has provided Carbone with at least fourteen (14) days' notice of a need to reschedule the Event, (ii) the rescheduled Event is to occur within seven (7) days of the original date, and (iii) a rescheduling fee has not already been included in the Compensation, a rescheduling fee in the amount of fifty percent (50.0%) of the Compensation will be immediately due and payable to Carbone.</li>
                                            <li><b><span class="underline">OVERTIME.</span> THE SERVICES SHALL BE PERFORMED FOR THE NUMBER OF HOURS/MINUTES SET FORTH ON THE SUMMARY SHEET. CUSTOMER UNDERSTANDS THAT, IF HE/SHE/IT DESIRES FOR THE SERVICES CONTINUE BEYOND THE ALLOTTED DURATION, CUSTOMER WILL BE REQUIRED AT THAT TIME TO AUTHORIZE ACCRUAL OF ADDITIONAL CHARGES, THROUGH THE CESSATION OF SUCH SERVICES, AT THE INCREASED "OVERTIME" RATE SET FORTH ON THE SUMMARY SHEET.</b></li>
                                            <li><span class="underline">Payment of Balance; Expenses.</span> On or before the "Balance Due Date" set forth on the Summary Sheet, Customer shall remit to Carbone the unpaid balance of the Compensation, plus the full amount of Overtime, if any (collectively, the "Balance"). In addition, within {!Event__c.Actual_Payment_Terms__c} days of Carbone's presentation to Customer of an itemized statement of reasonable expenses incurred by Carbone in connection with, or related to, the performance of the Services, Customer shall remit full payment of such expenses.</li>
                                            <apex:outputText rendered="{!IF(Event__c.Remove_Credit_Card_Payment_Option__c=FALSE,TRUE,FALSE)}">
                                                <li><span class="underline">Method of Payment.</span> Customer agrees to remit each payment by either (i) personal check payable to "Carbone Entertainment, Inc."; or (ii) credit card (Visa, MasterCard, or Discover).</li>
                                            </apex:outputText>
                                            <apex:outputText rendered="{!IF(Event__c.Remove_Credit_Card_Payment_Option__c=TRUE,TRUE,FALSE)}">
                                                <li><span class="underline">Method of Payment.</span> Customer agrees to remit each payment by personal check payable to "Carbone Entertainment, Inc.".</li>
                                            </apex:outputText>                                                     
                                            <li><span class="underline">Returned Checks; Late Payments.</span> Each check that is returned for insufficient funds shall incur a fee in the amount of $45 and, in the case of a deposit, shall be ineffective to reserve a date for the Event. Late payment of the Balance or of reimbursements shall result in the imposition of (i) a late fee in the amount of $100 and (ii) accruing simple interest on the unpaid amount at the rate of 6% per annum or the highest rate permitted by law, whichever is lesser. In the event that any amount remains outstanding for 60 days after its due date, the interest rate on such unpaid amount shall prospectively increase to 10% per annum or the highest rate permitted by law, whichever is lesser.</li>
                                        </ol>   
                                    <span class="number">4.<span class="ordered-list"><span class="underline">Independent Contractor Relationship.</span> Carbone is an independent contractor, and nothing in this Agreement shall render Carbone or any of its agents, employees or subcontractors an employee, partner, agent, or joint venturer of/with Customer for any purpose. Neither Party is authorized to act as agent or bind the other Party except as expressly stated in this Agreement.</span></span>
                                    <span class="number">5.<span class="ordered-list"><span class="underline">Method of Performing Services; Results.</span> In accordance with Customer's objectives, Carbone will determine in its sole discretion the method, details and means of performing the Services. Customer will have no right to, and shall not, control the manner or determine the method of performing Carbone's Services. Carbone shall provide the Services to the reasonable satisfaction of Customer.</span></span>
                                    <span class="number">6.<span class="ordered-list"><span class="underline">Subcontractors; Non-Solicitation.</span> Carbone has the right to hire and/or use subcontractors and/or employees in the performance of the Services and to charge Customer for such persons' time in accordance with the Summary Sheet. For a period of eighteen (18) months of full performance of the Services, Customer shall not directly or indirectly, without Carbone's prior written consent, (i) enter into any negotiations or transactions with any person, entity or affiliate, or (ii) solicit, hire or influence, any subcontractor, employee or agent, where any such contact was disclosed, introduced or otherwise revealed to Customer by Carbone.</span></span>
                                    <span class="number">7.<span class="ordered-list"><span class="underline">Nonexclusive Services.</span> Carbone may, during the term of this Agreement, render services on its own account or for any other person or entity as Carbone, in Carbone's sole discretion, sees fit. Customer and Carbone each agree that there is no exclusivity in the arrangements contemplated hereunder.</span></span>
                                    <span class="number">8.<span class="ordered-list"><span class="underline">Confidentiality.</span></span></span> 
                                        <ol class="sub">
                                            <li>For purposes of this Agreement, "Confidential Information" means information concerning a Party or a third party that is disclosed to another Party and generally not known by the public at large, including, but not limited to information concerning its business, financial condition, operations, marketing and public relations strategies, systems of operations, employee or contractor information, techniques, or trade secrets. Confidential Information does not include (i) information that is in the public domain, (ii) information that becomes public knowledge through no fault of the receiving Party or any wrongful disclosure by others, (iii) information of which a Party already was aware prior to disclosure by the other Party, which fact can be shown by reasonable demonstrable evidence; or (iv) the fact that Carbone is providing the Services to Customer.</li>
                                            <li>In connection with the engagement contemplated in this Agreement, one Party may disclose Confidential Information (hereinafter defined) to another Party, which the receiving Party agrees it will not disclose or reveal to any other person or entity, except that (1) Carbone may do so, in its reasonable discretion, where necessary for purposes of rendering the Services; and (2) a Party may do so as required by law, but only to the extent so required.</li>
                                        </ol>
                                    <span class="number">9.<span class="ordered-list"><b><span class="underline">LIMITATION OF LIABILITY.</span> CARBONE SHALL NOT BE LIABLE FOR ANY INDIRECT, SPECIAL, PUNITIVE, EXEMPLARY, INCIDENTAL OR CONSEQUENTIAL DAMAGE OR LOSS HOWSOEVER ARISING. IN NO EVENT SHALL CARBONE'S MAXIMUM LIABILITY HEREUNDER EXCEED THE AMOUNT ACTUALLY PAID TO CARBONE UNDER THIS AGREEMENT. THIS WAIVER INCLUDES BUT IS NOT LIMITED TO LOSS OF USE, LOSS OF PROFITS, LOSS OF INCOME, LOSS OF REPUTATION, UNREALIZED SAVINGS OR DIMUNITION OF PROPERTY VALUE. THE LIMITATION SET FORTH HEREIN IS FOR ANY AND ALL MATTERS FOR WHICH CARBONE MAY OTHERWISE HAVE LIABILITY ARISING OUT OF OR IN CONNECTION WITH THIS AGREEMENT, WHETHER THE CLAIM ARISES IN CONTRACT, TORT, STATUTE OR OTHERWISE, INCLUDING BUT NOT LIMITED TO NEGLIGENCE, STRICT LIABILITY, BREACH OF CONTRACT AND BREACH OF WARRANTY ON THE PART OF CARBONE OR ANY SUBCONTRACTOR, EMPLOYEE OR OTHER AGENT.</b></span></span>                    
                                    <span class="number">10.<span class="ordered-list"><span class="underline">Representations and Warranties.</span></span></span>
                                        <ol class="sub">
                                            <li><span class="underline">Compliance with Laws.</span> Customer hereby warrants and represents that he/she/it is in material compliance with all applicable laws, statutes, rules, regulations and ordinances relating to the Services and will remain so throughout its engagement of Carbone.</li>
                                            <li><span class="underline">Conflicts of Interest.</span> Each Party represents to the other Party that it is free to enter into this Agreement and that this engagement does not violate the terms of any agreement between it and any third party. Each Party shall defend, indemnify and hold the other Party and its successors, permitted assigns and licensees harmless from any and all claims, actions and proceedings, and the resulting losses, damages, costs and expenses (including reasonable attorney's fees) arising from any claim, action or proceeding based upon or in any way related to such Party's breach of any representation, warranty or covenant contained in this subsection.</li>
                                            <li><b><span class="underline">DISCLAIMER OF WARRANTIES.</span> THE WARRANTIES CONTAINED HEREIN ARE THE ONLY WARRANTIES MADE BY THE PARTIES HEREUNDER. EACH PARTY MAKES NO OTHER WARRANTY, WHETHER EXPRESS OR IMPLIED, AND EXPRESSLY EXCLUDES AND DISCLAIMS ALL OTHER WARRANTIES AND REPRESENTATIONS OF ANY KIND, INCLUDING ANY WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND TITLE. </b></li>
                                        </ol>
                                    <span class="number">11.<span class="ordered-list"><span class="underline">Miscellaneous.</span></span></span> 
                                        <ol class="sub">
                                            <li><span class="underline">Force Majeure.</span> Carbone shall not be liable for delay or failure in performance of the Services if such delay or failure is caused by conditions beyond its reasonable control. Carbone shall notify Customer of the occurrence of such an event as soon as reasonably practicable.</li>
                                            <li><span class="underline">Marketing; Publicity.</span> Unless and until Customer notifies Carbone in writing to the contrary, Customer hereby consents to Carbone's use of Customer's name in any advertising or promotional literature or website, and Carbone may advertise or refer publicly to the retention of Carbone for the Services. In addition, Customer hereby consents to Carbone's use, for advertising or promotional purposes, of photographs, videos, and other images that are both (i) created by or for Carbone, and (ii) relating to the Services.</li>
                                            <li><span class="underline">Intellectual Property.</span> The provision of any information, data, content, intellectual property or other asset to the other Party under this Agreement shall not affect the ownership of such information, data, content, intellectual property, know-how, work product or other asset. Each Party acknowledges and agrees that any materials used and/or provided by another Party in the course of rendering Services shall not be used by the other Party, other than in the provision of the Services, without such Party's specific written consent. </li>
                                            <li><span class="underline">Assignment.</span> Neither Party may assign any of his/her/its rights under this Agreement, without the prior written consent of the other Party, except that Carbone may freely assign this Agreement in its entirety (or any portion of it) to an affiliate.</li>
                                            <li><span class="underline">Successors and Assigns;</span> No Third Party Beneficiary Rights. All of the provisions of this Agreement shall be binding upon and inure only to the benefit of the Parties and their respective heirs, if any, permitted successors, and permitted assigns. No provision of this Agreement shall in any way inure to the benefit of any third party (including the public at large) so as to render any such person a third party beneficiary of this Agreement or any provision hereof, or otherwise give rise to any cause of action in any person not a Party.</li>
                                            <li><span class="underline">Indemnification.</span> Customer shall defend, indemnify, and hold Carbone and its officers, employees, agents, representatives, affiliates, successors, assigns and licensees harmless from any and all claims, actions, and proceedings, and the resulting losses, damages, costs and expenses (including reasonable attorneys' fees) arising from any claim, action or proceeding based upon or in any way related to the negligence, intentionally tortious conduct, or breach of this Agreement by Customer or any of Customer's employees, independent contractors, agents, or guests, as the case may be, including but not limited to conduct that causes physical damage to equipment/belongings of Customer's subcontractors.</li>
                                            <li><span class="underline">Governing Law; Venue.</span> The construction, interpretation, and performance of this Agreement shall be governed by and construed in accordance with the laws of the State of Maryland, without regard to conflicts of laws principles. Except as otherwise provided in this Agreement, any dispute arising from or related to this Agreement shall, to the extent subject matter jurisdiction otherwise exists, be resolved in a Maryland state court located in Montgomery County or in the United States District Court for the District of Maryland, Southern Division. Accordingly, with respect to any such court action, Contractor (i) submits to the personal jurisdiction of such courts; (ii) consents to service of process outside the State Maryland, if applicable; and (iii) waives any other requirement (whether imposed by statute, rule of court, or otherwise) with respect to personal jurisdiction or service of process.</li>
                                            <li><span class="underline">Disputes.</span> In the event of any dispute, other than a dispute for nonpayment, the Parties agree to endeavor to resolve any such matter through good faith discussions for a period of at least fifteen (15) days prior to filing a lawsuit.</li>
                                            <li><span class="underline">Waiver.</span> Waiver by one Party of breach or default of any provision of this Agreement by the other shall not operate or be construed as a continuing waiver, nor shall any delay or omission on the part of either party to exercise or avail itself of any right, power or privilege that it has or may have hereunder operate as a waiver of any breach or default.</li>
                                            <li><span class="underline">Notices.</span> All notices and consents must be in writing and will be deemed effective when received by (i) registered mail; (ii) certified mail, return receipt requested; (iii) overnight mail; (iv) courier; (v) email; or (vi) facsimile. Mail and delivery shall go to the mailing addresses listed on the Summary Sheet, and emails/facsimiles shall go to the addresses/numbers listed on the Summary Sheet. Notwithstanding the foregoing, either Party may change the address/email/fax number to which notices to him/her/it are to be sent by providing written notice to the other Party as provided in this subsection. </li>
                                            <li><span class="underline">Modification or Amendment.</span> No amendment, change or modification of this Agreement shall be valid unless in writing signed by the Parties.</li>
                                            <li><span class="underline">Entire Understanding.</span> This Agreement, including the fully incorporated the Summary Sheet, constitutes the entire understanding and agreement of the Parties, and any and all prior agreements, understandings, and representations are hereby superseded, terminated and canceled in their entirety and are of no further force and effect.</li>
                                            <li><span class="underline">Unenforceability of Provisions.</span> If any provision of this Agreement, or any portion thereof, is held to be illegal, invalid and unenforceable, then the remainder of this Agreement shall nevertheless remain in full force and effect. Furthermore, if the scope of any provision of this Agreement is determined to be too broad in any respect whatsoever to permit enforcement to its maximum extent, then such provision shall be enforced to the maximum extent permitted by law.</li>
                                            <li><span class="underline">Survival.</span> The Parties acknowledge and agree that Sections 4-11 shall survive this Agreement regardless of the reason, basis or circumstances of the termination of this Agreement.</li>
                                            <li><span class="underline">Section Headings.</span> Section and subsection headings are inserted for convenience only and are not intended to affect the meaning or interpretation of this Agreement.</li>
                                            <li><span class="underline">Executed Counterparts; Electronic Signatures.</span> This Agreement may be executed in any number of counterparts, and all counterparts shall be considered together as one agreement. The Parties agree that electronic, digital or PDF signatures submitted via Carbone's website shall be as effective as if originals.</li>                        
                                        </ol>
                                        <p>IN WITNESS WHEREOF the undersigned have executed this Agreement as of the day and year first written above.</p>
                                    </div>
                                </apex:outputPanel>
                 </apex:column>
             </apex:pageblockTable>   
          
           <apex:pageblockTable value="{!Event__c}" var="a" columnswidth="360,360" style="font-size:.8em;">
               <apex:column style="text-align:center;vertical-align:top;">          
                   <b>CARBONE ENTERTAINMENT, INC.</b><br />
                   <apex:image value="{!$Resource.Karen_Sig2}" height="70"/><br />
                   Karen Carbone<br />
                   Owner and CEO
               </apex:column>       
               <apex:column style="vertical-align:top;">
                   <div style="text-align:center;margin-bottom:0;"><b>{!UPPER(Event__c.Account__r.Name)}</b></div>
                   <span style="display:block;text-align:center;margin-bottom:0;"><a href="{!Event__c.Sign_Contract_Link__c}">Save paper, Sign electronically.</a></span>
                   <span style="display:block;height:40px;width:100%;border-bottom:1px solid black;"/>
                   <div style="text-align:left;padding-top:5px;width:100%;margin:auto;">Name:</div>
                   <div style="text-align:left;padding-top:10px;width:100%;">Title:</div>                  
               </apex:column>                            
           </apex:pageblockTable>                  
        </apex:pageBlock>                        
                
    </body>
</html>
</apex:page>
```


The contract is sent via a button that calls the following Apex Class. Once the contract has been sent, a copy of the PDF document is attached to the record and a custom field is updated with the timestamp of the action.


```
public class SendContractEmail {
    private final Event__c event;
    // Create a constructor that populates the Event__c object
    public sendContractEmail() {
        event = [SELECT Name , Client__r.Email , Contract_Sent__c FROM Event__c WHERE id = :ApexPages.currentPage().getParameters().get('id')];
    }
    public Event__c getEvent() {
        return event;
    }   
    public PageReference send() {
        // Define the email        
        Messaging.SingleEmailMessage email = new Messaging.SingleEmailMessage(); 
        // Sets the paramaters of the email
        email.setTargetObjectId( event.Client__r.ID );
        email.setTemplateID( 'insert template id' );
        email.setWhatID( event.ID );
        // Reference the attachment page, pass in the account ID
        PageReference pdf = Page.Contract_PDF;
        pdf.getParameters().put('id',(String)event.id);
        pdf.setRedirect(true);
        // Take the PDF content
        Blob b;
            if (Test.IsRunningTest()){
            b=Blob.valueOf('UNIT.TEST');
            }else{
            b= pdf.getContent();
            }
        // Create the email attachment
        Messaging.EmailFileAttachment efa = new Messaging.EmailFileAttachment();
        efa.setFileName('Carbone Entertainment Contract for '+event.Name+'.pdf');
        efa.setBody(b);
        email.setFileAttachments(new Messaging.EmailFileAttachment[] {efa});    
        // Sends the email
        Messaging.SendEmailResult [] r = Messaging.sendEmail(new Messaging.SingleEmailMessage[] {email});
        // Attach the PDF to Event__c
        Attachment attach = new Attachment();
        Blob c;
            if (Test.IsRunningTest()){
            c=Blob.valueOf('UNIT.TEST');
            }else{
            c= pdf.getContent();
            }
        attach.Body = c;
        // add the user entered name
        attach.Name = 'Carbone Entertainment Contract for '+event.Name+'.pdf';
        attach.IsPrivate = false;
        // attach the pdf to the account
        attach.ParentId = event.id;
        insert attach;                          
        // Update "Event__c.Contract_Sent__c" with time stamp
        event.Contract_Sent__c = DateTime.NOW();
        update event; 
        // Close window after sending
        return new PageReference('javascript:window.close()'); 
    }
}
```


The recipient will receive an email that is designed with a Visualforce Email Template. This email template utilizes the [Foundation for Emails](https://get.foundation/emails.html) framework from ZURB.

 ![Contract email template](/img/contract-email-template.png)
