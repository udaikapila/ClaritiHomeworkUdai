
# Udai Kapila - Clariti Interview Assignment

**This project contains the custom code for Field Change History functionality. This functionality will allow the user to track field changes, compare old values to new values and to filter on the change history by Object, Field Name and Change Date. The functionality has 4 main components: Metadata config to choose the Object and Fields to Track, an invocable apex class that will create a new Field_Change_History__c record, a flow to trigger history tracking, and a LWC to display and filter history records.**
&nbsp;   
&nbsp;

# How it works

Once set up, the Field Change History functionality works as such:

1. Flow to trigger apex: To gauge the object and get the records that we are tracking, the user will set up a flow to trigger on update (only) of the Object that they want to track.
2. Metadata: a Custom Metadata Type will be used to store a list of the Object and the Fields on that Object to track. The list of fields is comma separated to allow a large number of fields to be tracked and easily edited. 
3. Once the flow and custom metadata are both configured, when a record from a tracked object is created, the FieldHistoryCreateRecordHelper class is used to create a new record in a Custom Object called "**Field_Change_History__c**". 
4. The helper class will first get the set of fields from the metadata, and check if these fields have been changed on the triggered records, if so, it will create a new Field_Change_History__c record. 
5. Once the record has been created in Field_Change_History__c, we can use it in Reports, Dashboards or as shown in a custom LWC called fieldHistoryComponent that allows us to filter and search the object for any and all changes.


# Setup
To walk through set up, let's use the Contact object as an example.

## Step 1 : Create Custom metadata type record
1. Go to Setup > Custom Metadata Types and click "**ManageRecordHistoryTracking**" custom metadata type.
2. Create new record by clicking "**Manage ManageRecordHistoryTracking**" and then "**New**" button.
3. Enter any appropriate label like "**Contact Field Tracking**" and accept default "**ManageRecordHistoryTracking Name**" and select Contact in the dropdown under Tracking object. Under "**Tracked Fields List**" add the list of fields to track separated by a comma "**,**". For example "**Name,Phone,Email**". These should match the API Name of the field exactly. Save the record.

## Step 2: Create a flow 
1. create new record triggered lightning flow on the Object you'd like to track, in this case, "**Contact**". Select "**A record is updated**" with all other default settings.
2. Add an element after the and select apex action, under the action select "**insert history record**" action. 
3. Give any appropriate name to action. Under newRecordState and  oldRecordState for input and output select the "**Contact**" object on all 4 fields.
4. Under "**Set Input Values**" section for newRecordState put {!$Record}  and on oldRecordState put {!$Record__Prior}.
5. Save and activate the flow.

## Step 3 : Assign Permission Set to users
1. Go to setup and Permission Sets and find the 'Field History Tracking' permission set. 
2. Click on "**Manage Assignments**" and then "**Add Assignment**" button.
2. Select the users you'd like to give access to.

Note: If this is not included, field history records will not be created, and the LWC will show up as a blank screen.

## Step 4 : Add the Lightning Web Component to your page
1. Go to the page that you'd like to add the filtered history list to, click on the cog in the top right of the page, and click on "**Edit Page**". 
2. On the Lighting Page layout editor, find the custom component called "**fieldHistoryComponent**". Drag it to where you'd like to place it on the page. 
3. Save and Activate the page. 


# Considerations, Assumptions, Future Enhancements

1. We are assuming that the user knows the Object and Field API Names when entering Metadata configs. 

2. When storing records, there were multiple options considered:
 - Storing changes in a JSON file instead of creating a record for each field change. This would help group changes made to one record into a single Field_Change_History__c record, saving storage space. However, we would have to use custom reports with development. Since reporting was considered an MVP, this method was not used.
 - Batching and storing changes in a Big Object. This is a little more user friendly in that we have separate records for each field change. This would also remove the storage considerations from the org limits. However, Big Objects are not supported with standard Salesforce Reporting. Furthermore, batching changes to update the Big Object would create a significant delay in the records being inserted and available. 
 - Installing an App from the App Exchange, such as this [Enhanced Field Tracking App](https://appexchange.salesforce.com/appxListingDetail?listingId=6d0f039b-bd03-4ffb-9b5d-9bee5080e7cc). The obvious upside of using an existing app is that no development is needed. The drawback of this is that customizing the app for specific functionalities will be restricted and not in our control.

 3. Data Considerations: Creating records on "Only Update" and not on new records will save us unnecessary records. However, if storage is a major consideration in the Org, then putting in a purge policy and an automated scheduled job job to remove records from the History table would help. 
 - Add settings to the ManageRecordHistoryTracking metadata for "Number of Days", to indicate the number of days to keep this History Record.
 - Add a nightly schedled job to find all expired records and delete them. I would also add any records that do not fit the field list criteria anymore. 

4. LWC Consideration: I debated 2 different approaches to the LWC. For the datatable, do we fetch and display the filtered data from the server on every change, or do we have a subset of the most recent records and filter on the client side only. Thinking about the use case for a large number of records, I went with the server side retrieval. This would ensure that all History records are searchable and served up to the user. However I did build a "Client Side Only" version and happy to demo it.

5. LWC Enhancements: 
- Replace Ids with the name of the Lookup record. This would make the filter table a lot less messy and more usable.
- Add a LWC that would filter based on the Record so that we can add a related list type view to a Lightning Record Page.
- Add a "Revert" feature, that woudl revert the new value to the old value.

# Last Thoughts

I spent the first few days of the week thinking about the data structure itself. I think that Salesforce development is trending towards more JSON heavy dependencies and a JSON Event log can be potentially very useful. This also opens up the window to send Event Changes in a JSON to an external database or a BI Tool. 

When coming to the actual implementation, I considered using Platform Events and subscribing to changes on specific objects. I gathered from the hint given that this is probably a solution. The solution in my mind would have been something involving using a LWC to trigger a Platform Event on a particular object to trigger a History record creation. I considered this but eventually concluded that it was a bit of a hacky approach. I do think it's possible to be achieved. 

The other consideration was to create a trigger handler utility method that would be invoked on update of a trigger by default. However this would involve creating and deploying a trigger for every Object that we want to track.

All in all, this was a fun assingment. I definitely spent more than a couple of hours on it and I relied heavily on the SFDC Community for tips, tricks and ideas. Hopefully, this solution is somehwere in the ballpark of what you were looking for! 

Thanks,

Udai.
