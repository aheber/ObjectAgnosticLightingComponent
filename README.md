
# Building Object Agnostic Lightning Components
Instructions will assume you're using the Developer Console, feel free to translate to your favorite editor.

The structure of this doc adds code incrementally to your project. This doesn't work for everyone because sometimes you have a hard time with spacial awareness in code, I know that I do. Along the way I've added lots of collapsed sections which include the full component code to that point. If you get lost or something doesn't work just grab the nearest "Full Component" and paste it in.


## Setup
This is all done using Lightning Experience.

Make sure you have a developer edition available to you. Your org will need to have My Domain enabled, this is required to develop custom Lightning Components. Trailhead orgs are often ready to go for this.
[Setup My Domain](https://trailhead.salesforce.com/en/projects/slds-lightning-components-workshop/steps/slds-lc-1)

ONLY IN A DEVELOPMENT ENVIRONMENT Disable caching of components.

Setup -> Session Settings --> Caching --> uncheck _Enable secure and persistent browser caching to improve performance_

This will make sure when you display the page you get the most recent version of your component as you're building it.

## Phase 1

### Create a new Lighting Component that can be placed on a record
Using Dev Console select New -> Lightning Component.
Give it a name of RecordInfo, select the Lightning Record Page configuration.
It should look something like this
```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId" access="global" >
	This is the Record Info component
</aura:component>
```
 You can see that by selecting Lighting Record Page it has added some _implements_ attributes, we call these interfaces, this is the first way we're letting this component know about its surroundings.
 * flexipage:availableForRecordHome - tells the system that this component can be placed on a record page rather than a home page, mobile page, etc...
 * force:hasRecordId - tells the system that this component is ready to receive the Id value of the record it is placed on, we'll use this to get information from this record for display

You can think of an interface by thinking of an international plug adapter, your component is the adapter and the various plug types are the interfaces that you implement. You're describing ways that the Lighting framework can plug into your component.

For the sake of future display improvements, lets add a Style file. With the component open in Developer Console click on STYLE in the right column. Add the following.
```css
.THIS label {
    font-weight: bold;
}
```

### Add it to the Account record page
Navigate to an Account, using the gear at the top of the page select Edit Page.
Once in the Page Builder scroll down the component list to the Custom section, grab the RecordInfo component and place it on the top-right of the Account Page you're building. Save and, if necessary, activate as the Org Default.


### Teach the Lighting Component about the record it is displayed on
Let start by displaying what data we have available now.
Lets add the record id to the component output so we can make sure we can see it

```html
Record Id: {!v.recordId}
```

<details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId" access="global" >
	Record Id: {!v.recordId}
</aura:component>
```

</p>
</details>
  
Go ahead and refresh the account page, you should see the record id floating on the background.

So far our component implements gathering the record id but there is one piece we want to add that will help us get some additional information about our record. Lets add the _force:hasSObjectName_ inteface to the list of interfaces that we implement.

Your component line should look like this now

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global">
```

We're going to display the data we get from that new interface right below the record id

```html
SObject Name: {!v.sObjectName}<br />
```

<details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global">
	Record Id: {!v.recordId}<br />
    SObject Name: {!v.sObjectName}<br />
</aura:component>
```

</p>
</details>
  
At this point we're doing very little. If you wanted the record id you'd pull it from the address bar and if you're looking at an account record and don't know it is an account record, you might be beyond help.

That said, we are on track. You can take this component and place it on any record in your whole system and it would work, as nearly worthless as that is... but we're getting there.

### Add Lighting Data Service to the component

The first piece of value we're going to add will be Lightning Data Service. Lightning Data Service is designed to allow a component easy access to record data. Finally we can get some real work done without Apex! Lightning Data Service implements a cache strategy to help reduce trips to the server, if we ask it for data it will get it for us, either from the cache held on the client, or by going to the server for the data. Luckily we don't have to worry about how it does it, we just ask for the data and the data magically appears. Lightning Data Service also handles security, if your user shouldn't see the field data they won't see the data.

OK, that was a lot of talking, more code please!

To add Lighting Data Service we just need a few more lines in our component.
We'll add attributes to hold the record itself, the field values, and if needed any errors.
To make the future easier we're also going to add an array of values for the specific fields we want to get from the database.

We'll add the force:recordData component to our system, this is the connection to Lighting Data Service. We link all the attributes we've added to the component so we can use it later.

```html
    <aura:attribute name="record" type="Object"/>
    <aura:attribute name="simpleRecord" type="Object"/>
    <aura:attribute name="recordError" type="String"/>


    <!-- record that we're sitting on -->
    <aura:attribute name="recordfields" type="String[]" default="Name,CreatedBy.Name,CreatedDate,OwnerId"/>
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      />
```

Below the record id and sobject name we'll output some data from Lighting Data Service. We'll start using a little more HTML for styling and structure.

```html
	<!-- Record Name -->
    <label>Name:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.Name}"/><br />
    <!-- CreatedDate, CreatedBy -->
    <label>Created By:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.CreatedBy.Name}"/>&nbsp;on
    <lightning:formattedDateTime value="{!v.simpleRecord.CreatedDate}" year="2-digit" month="short" day="2-digit" weekday="long"/>
    <br />

```

Below that, at the bottom of the component, output the errors if needed.

```html
	<!-- Display Lightning Data Service errors, if any -->
    <aura:if isTrue="{!not(empty(v.recordError))}">
    	<div class="recordError">
        	<ui:message title="Error" severity="error" closable="true">
				{!v.recordError}
			</ui:message>
		</div>
	</aura:if>
```

<details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global">
    
    <aura:attribute name="record" type="Object"/>
    <aura:attribute name="simpleRecord" type="Object"/>
    <aura:attribute name="recordError" type="String"/>
    
    
    <!-- record that we're sitting on -->
    <aura:attribute name="recordfields" type="String[]" default="Name,CreatedBy.Name,CreatedDate,OwnerId"/>
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      />
    
    
    Record Id: {!v.recordId}<br />
    SObject Name: {!v.sObjectName}<br />
    
    <!-- Record Name -->
    <label>Name:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.Name}"/><br />
    <!-- CreatedDate, CreatedBy -->
    <label>Created By:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.CreatedBy.Name}"/>&nbsp;on
    <lightning:formattedDateTime value="{!v.simpleRecord.CreatedDate}" year="2-digit" month="short" day="2-digit" weekday="long"/>
    <br />
    
    <!-- Display Lightning Data Service errors, if any -->
    <aura:if isTrue="{!not(empty(v.recordError))}">
        <div class="recordError">
            <ui:message title="Error" severity="error" closable="true">
                {!v.recordError}
            </ui:message>
        </div>
    </aura:if>
</aura:component>
```

</p>
</details>

## Phase 2

### Learn about Events

Lighting is an event driven system. Lots of components on the page and they communicate by sending messages. Some of those messages are the component talking to itself and some of them are messages going out for the world to hear.

To see messages in action, change the name of the account record. When you change the name on the detail section of the page it will send a message for the world to hear, the force:recordData component will grab that message and it will reload the force:recordData values which in turn sends another message that the attributes linked to it have changed and whereever you've output that data in your component will be updated because it is listening for just that sort of event.

For the purposes of this tutorial we're mostly going to consume messages sent to us from the framework. The first one we'll want to capture is knowing when Lighting Data Service has our data ready.  


### Use Events to respond to changes

To get this working we will finally need to use our controller, in the Developer Console on the right side, click Controller in the bundle to create one. It will start with a basic method already defined named _myAction_. We're going to kill that and replace the name _myAction_ with _handleRecordUpdated_.

We'll make the controller throw up a message whenever the record is loaded. Here is the full controller we're starting with.

```javascript
({
	handleRecordUpdated : function(component, event, helper) {
		alert('Record Loaded');
	}
})
```

Lets add an attribute to force:recordData to define a controller method to be called when the record is updated.

The force:recordData should now look like this

```html
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.handleRecordUpdated}"
                      />
```
  

<details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global">
    
    <aura:attribute name="record" type="Object"/>
    <aura:attribute name="simpleRecord" type="Object"/>
    <aura:attribute name="recordError" type="String"/>
    
    
    <!-- record that we're sitting on -->
    <aura:attribute name="recordfields" type="String[]" default="Name,CreatedBy.Name,CreatedDate,OwnerId"/>
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.handleRecordUpdated}"
                      />
    
    
    Record Id: {!v.recordId}<br />
    SObject Name: {!v.sObjectName}<br />
    
    <!-- Record Name -->
    <label>Name:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.Name}"/><br />
    <!-- CreatedDate, CreatedBy -->
    <label>Created By:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.CreatedBy.Name}"/>&nbsp;on
    <lightning:formattedDateTime value="{!v.simpleRecord.CreatedDate}" year="2-digit" month="short" day="2-digit" weekday="long"/>
    <br />
    
    <!-- Display Lightning Data Service errors, if any -->
    <aura:if isTrue="{!not(empty(v.recordError))}">
        <div class="recordError">
            <ui:message title="Error" severity="error" closable="true">
                {!v.recordError}
            </ui:message>
        </div>
    </aura:if>
</aura:component>
```

</p>
</details>
  
  
### Add a second instance of Lighting Data Service

We're going to start making things more dynamic. For fun we're going to add another force:recordData component. This will start with a blank record id so it won't actually load yet, it is going to sit around and wait for someone to tell it what to do.

Using our handleRecordUpdated controller method, when the first record is loaded we'll assign the OwnerId from our base record on to the second so we can display some additional data. To make this work we'll have to get the OwnerId data from the first and set it onto the second's id value. We'll then fire an event on the component to tell it to go get its data from Lighting Data Service.

Change the Controller to match this
```javascript
({
    handleRecordUpdated: function(component, event, helper) {
        var eventParams = event.getParams();
        if(eventParams.changeType === "LOADED" || eventParams.changeType === "CHANGED") {
            // base record is loaded from Lighting Data Service

			// get the simpleRecord attribute from the component
			var simpleRecord = component.get('v.simpleRecord');
            
            var ownerId = component.get('v.recordOwnerId');
            // if this is the first time we're loading the record then 
            // populate the ownerid and load the record
            if(ownerId == null){
                // assign the base record's ownerid onto the attribute we've
                // attached to the second force:recordData's recordid value.
                component.set('v.recordOwnerId', simpleRecord.OwnerId);
                // fire the component event to reload the record now
                // that we've given it a record id.
                component.find('recordOwner').reloadRecord(true);
            }
        } else if(eventParams.changeType === "CHANGED") {
            // record is changed
        } else if(eventParams.changeType === "REMOVED") {
            // record is deleted
        } else if(eventParams.changeType === "ERROR") {
            // there’s an error while loading, saving, or deleting the record
        }
    }

})
```

Add the second force:recordData along with some attributes to hold the data

```html
    <aura:attribute name="recordOwnerId" type="Id"/> <!-- assigned in Javascript after record load -->
    <aura:attribute name="recordOwner" type="Object"/>
    <aura:attribute name="simpleRecordOwner" type="Object"/>
    <aura:attribute name="recordErrorOwner" type="String"/>
    
	<!-- second instance of recordData component to get related record information -->
    <!-- owner of the record -->
    <force:recordData aura:id="recordOwner"
                      recordId="{!v.recordOwnerId}"
                      targetRecord="{!v.recordOwner}"
                      fields="Name,SmallPhotoUrl"
                      targetFields="{!v.simpleRecordOwner}"
                      targetError="{!v.recordErrorOwner}"
                      recordUpdated="{!c.handleRecordUpdated}"
    />
```

Add some output for the Owner information, including name and picture.

```html
    <!-- Owner Picture -->
    <label>Owner</label><br /><lightning:avatar src="{!v.simpleRecordOwner.SmallPhotoUrl}"
                                                fallbackIconName="utility:inbox" alternativeText="Salesforce"/>&nbsp;
    <!-- Owner Name --> <!-- what if owned by queue? --> <!-- what if owned by Master -->
    <ui:outputText value="{!v.simpleRecordOwner.Name}"/><br />
```

Now if you reload the page you'll see the record information along with the owner information. 

I know I'm making it complicated, really we could have just used relationship fields on the first componenet to display owner information, you would be more likely to use this if you wanted to be able to edit multiple records. Sometimes we take the long road just to showcase functionality and boggle your mind with possibilities.

## TAKE A BREAK
We're going to update the styling on our component real quick. It looks terrible! No background so you can't read the text, it doesn't look like it belongs in lightning at all.

Quickly copy the component from here to get the Lightning Design System injected into your component. Go ahead, I'll wait.


<details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global">
    
    <aura:attribute name="record" type="Object"/>
    <aura:attribute name="simpleRecord" type="Object"/>
    <aura:attribute name="recordError" type="String"/>
    
    <!-- record that we're sitting on -->
    <aura:attribute name="recordfields" type="String[]" default="Name,CreatedBy.Name,CreatedDate,OwnerId"/>
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.handleRecordUpdated}"
                      />
    
    <aura:attribute name="recordOwnerId" type="Id"/> <!-- assigned in Javascript after record load -->
    <aura:attribute name="recordOwner" type="Object"/>
    <aura:attribute name="simpleRecordOwner" type="Object"/>
    <aura:attribute name="recordErrorOwner" type="String"/>
    
    <!-- second instance of recordData component to get related record information -->
    <!-- owner of the record -->
    <force:recordData aura:id="recordOwner"
                      recordId="{!v.recordOwnerId}"
                      targetRecord="{!v.recordOwner}"
                      fields="Name,SmallPhotoUrl"
                      targetFields="{!v.simpleRecordOwner}"
                      targetError="{!v.recordErrorOwner}"
                      recordUpdated="{!c.handleRecordUpdated}"
                      />
    
    <article class="slds-card">
        <div class="slds-card__header slds-grid">
            <header class="slds-media slds-media_center slds-has-flexi-truncate">
                <div class="slds-media__body">
                    <h2>
                        <span class="slds-text-heading_small">Record Info</span>
                    </h2>
                </div>
            </header>
        </div>
        <div class="slds-card__body slds-card__body_inner">
            <!-- Owner Picture -->
            <label>Owner</label><br /><lightning:avatar src="{!v.simpleRecordOwner.SmallPhotoUrl}"
                                                        fallbackIconName="utility:inbox" alternativeText="Salesforce"/>&nbsp;
            <!-- Owner Name --> <!-- what if owned by queue? --> <!-- what if owned by Master -->
            <ui:outputText value="{!v.simpleRecordOwner.Name}"/><br />
            
            
            <!-- Record Name -->
            <label>Name:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.Name}"/><br />
            <!-- CreatedDate, CreatedBy -->
            <label>Created By:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.CreatedBy.Name}"/>&nbsp;on
            <lightning:formattedDateTime value="{!v.simpleRecord.CreatedDate}" year="2-digit" month="short" day="2-digit" weekday="long"/>
            <br />
            
            <!-- Display Lightning Data Service errors, if any -->
            <aura:if isTrue="{!not(empty(v.recordError))}">
                <div class="recordError">
                    <ui:message title="Error" severity="error" closable="true">
                        {!v.recordError}
                    </ui:message>
                </div>
            </aura:if>
        </div>
    </article>
</aura:component>
```
 
</p>
</details> 

<details><summary>Full Controller</summary>
<p>

```javascript
({
    handleRecordUpdated: function(component, event, helper) {
        var eventParams = event.getParams();
        if(eventParams.changeType === "LOADED" || eventParams.changeType === "CHANGED") {
            // base record is loaded from Lighting Data Service

			// get the simpleRecord attribute from the component
			var simpleRecord = component.get('v.simpleRecord');

            var ownerId = component.get('v.recordOwnerId');
            // if this is the first time we're loading the record then 
            // populate the ownerid and load the record
            if(ownerId == null){
                // assign the base record's ownerid onto the attribute we've
                // attached to the second force:recordData's recordid value.
                component.set('v.recordOwnerId', simpleRecord.OwnerId);
                // fire the component event to reload the record now
                // that we've given it a record id.
                component.find('recordOwner').reloadRecord(true);
            }
        } else if(eventParams.changeType === "REMOVED") {
            // record is deleted
        } else if(eventParams.changeType === "ERROR") {
            // there’s an error while loading, saving, or deleting the record
        }
    }
})

```
 
</p>
</details> 

## Phase 3

Our component is really starting to come together. We're using our interfaces to get the record id and SObject Name, then using that record id to get data from Lighting Data Service, then using that data to get more data about the owner. All of this is turning into a great way to... see what we already see on every record. The Name and the Owner. We'll start adding more to this very soon.

Our primary objective is to have a component that we can put on any object and it will work without error. We're already going to run into one problem with this. We've selected a few specific fields, these fields exist on most records but not all. Go ahead and add your component to the Case page.

See that error? `No such column 'Name' exists on entity 'Case'.`

This is a bit of a problem, doesn't everything have a Name field?! I guess not. So how do we get this component working on stuff that doesn't have a proper name?

For the Case record it has a name a just calls it something else. Case calls its name field CaseNumber. There are a few of the standard objects in Salesforce that use a different field name but serve the same purpose.

We're going to add the ability to tell our component the specific API name of the field that this object uses for its name value. That makes this component more flexible and will help it to function on the Case object.

### Add Configuration Attributes to the component’s Design file

If you've ever added a componenet to the page and it gave you the ability to provide specific parameters you've used Configuration Attributes. We're going to add a couple of these to our component so users can configure it the way they need.

Go ahead and add these attributes to your componenet file, these will act as variables to hold the configuration parameters that the user provides us.

```html
    <!-- config attributes -->
    <aura:attribute name="namefield" type="String" default="Name"/>
    <aura:attribute name="additionalfields" type="String"/>
```

Also, remove the Name field from the recordfields string array

```html
<aura:attribute name="recordfields" type="String[]" default="CreatedBy.Name,CreatedDate,OwnerId"/>
```


We are adding one attribute for the name field, we're also adding one for a list of other fields that should be displayed on the record so we can let the user customize this just a bit more.

Lets add another file to our Lighting Component Bundle, in Developer Console on the bundle click Design to have the design file added to the bundle and opened.

It should look like this when it opens up
```html
<design:component >
	
</design:component>
```

Not very interesting yet but we do love a blank canvas. While we are in this file we will actually set a display name for the component, not very fun when you go into the app builder and it shows RecordInfo without any spaces or anything. We'll go ahead and clean that up by adding a label value on the design. Inside we'll also add the design:attributes to enable data capture from the user, remember to set the name value the same as the attribute you added to the component.

The full Design file should now look like this
```html
<design:component label="Record Info">
    <design:attribute name="namefield" label="Name Field" />
    <design:attribute name="additionalfields" label="Additional Fields" />
</design:component>
```  

### Teach the component about its amazing new Configuration Attribute and use it to customize the output

So we finally have the Configuration Attributes added to our component but they don't do anything yet... you can edit the page, select the componenet, and put text in there, but it doesn't do anything. We should fix that.

Remember that we are using the Lightning Data Service to provide data, part of that is telling it what fields we need from the record. We have it hardcoded to get us the Name field, we already discussed how this is a problem.

This is where things like having our list of retrieved fields as a String array (recordfields attribute) becomes helpful, we can modify that array and get different fields.

The first thing is to attach to a new event, we want to know as soon as our component is ready for business. The framework throws us an _init_ event as soon as it has us ready to go and we can use this to do some of our setup work like adding to the list of fields to get from the record.

Inside our componenet we're going to add an aura:handler for the init event and point it at a controller methat that we'll also add.

Component
```html
<aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
```

Controller
```javascript
    doInit: function(component, event, helper){
        // merge fields into the query field set
        // get the default list of fields from the component
        var recordFields = component.get('v.recordfields');
        // add the Configuration Attribute that the user set to the list of
        // fields we want to pull back
        recordFields.push(component.get('v.namefield'));
        
        // Get the list of other fields the user wants to display
        // this should be a comma separated list of field API names
        // add those to the list of fields to get back from the component
        var additionalFields = component.get('v.additionalfields');
        
        // if the user specified additional fields then split them on the comma
        // and add them to the list of fields to get from Lightning Data Service
        if(additionalFields != undefined){
            // create a list of
            var splitAdditionalFields = additionalFields.split(',');
            // smash two lists into each other
            recordFields = recordFields.concat(splitAdditionalFields);
        }
        // put he updated list of fields back onto the componenet
        component.set('v.recordfields',recordFields);
    },
```

Go back and edit your case page in the App Builder. Select the component on the page and add data to the Configuration Attribute text boxes

Name = `CaseNumber` and Additional Fields = `Status,Priority`

Refresh your page and see the glorious changes you just brought about.

### Actually displaying the field values you worked so hard for

WHAT?! nothing happened? your Name field is still blank and you certainly don't see the additional fields you added?

At this point you've collected information from the user and added it to the array of fields. Let finish linking those things to our output.

A Lighting Component markup doesn't have the ability to dynamically grab a field value from the simpleRecord object so we need to take care of this in our javascript controller. The first thing we'll wire up is capturing the value from the dynamic name field.

We need a new aura:attribute to hold the string from whatever we called the name field, we'll also need an aura:attribute to hold the list of additional fields we want to display

In your componenet add the following
```html
	<!-- config attribute output helpers -->
	<aura:attribute name="namevalue" type="String"/>
	<aura:attribute name="additionalfielddata" type="Object[]"/>
```

Now change out this line

```html
	<label>Name:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.Name}"/><br />
```
with
```html
    <label>Name:</label>&nbsp;<ui:outputText value="{!v.namevalue}"/><br />
```

You can see that we don't pull the name value from the simpleRecord anymore, we pull it from our own attribute which unfortunately is still blank. We'll assign a value to in our controller. So onto the controller.

Back in the controller file find the _handleRecordUpdated_ method, we want to add some logic that once the Lighting Data Service fires the event to let us know that the record has data we can go and pilfer the value that we are assigning to the name and add it to our variable.

Add right under the `var simpleRecord = component.get('v.simpleRecord');`

```javascript
  // capture name value
  var nameField = component.get('v.namefield');
  component.set('v.namevalue',simpleRecord[nameField]);
```

With all that in place you should be able to reload the Case page and now your Name field should display the case number. You have your first dynamic field! More importantly you have this component back to working on any record you want to put it on.

We've forgotten about something though, remember the other configuration attribute we setup? We want to teach this componenent to display the configurable set of fields. We already did the work to add those fields to the Lighting Data Service request, they are now just waiting for us.

Lets add the aura:attribute to the component to hold the field data and use aura:iteration to loop through the fields and output the field name and value.

Add the aura:attribute
```html
    <aura:attribute name="additionalfielddata" type="Object[]"/>
```

Add the aura:iteration for output below the created by information
```html
  <aura:if isTrue="{!v.additionalfielddata.length > 0}">
  	<br /><span class="slds-text-title_caps">Additional Fields</span><br />
  	<aura:iteration items="{!v.additionalfielddata}" var="field">
  		<label>{!field.name}:</label>&nbsp;{!field.value}<br />
  	</aura:iteration>
  </aura:if>
```
You can see that we only display this section of the component if the user actually wants us to display more information. If they didn't give us any configuration then we supress the entire section via aura:if.

Now we modify the _handleRecordUpdated_ method to build the array of objects that should be displayed based on the users configuration.

Add this to the controller underneath `component.set('v.namevalue',simpleRecord[nameField]);`
```javascript

            // get the list of additional fields from the 
            // Configuration Attribute
            var additionalFields = component.get('v.additionalfields');
            var splitAdditionalFields;
            // If the user supplied any data
            if(additionalFields != undefined){
                splitAdditionalFields = additionalFields.split(',');
            }
            
            // get the attribute to hold the additional field data 
            var additionalfielddata = []
            
            // loop through defined fields, build an object and
            // push it onto the stack to be displayed by the component
            if(splitAdditionalFields != undefined){
                for(var i = 0; i < splitAdditionalFields.length; i++){
                    var field = splitAdditionalFields[i].trim();
                    var value = simpleRecord[field];
                    additionalfielddata.push({name:field,value:value});
                }
            }
            // push the data back to the component
            component.set('v.additionalfielddata',additionalfielddata);
```

Now you can refresh your Case page and you should see the Priority and Status fields as well as their values. It should also work that if you change one of those values on the record Detail page the component should refresh automatically by virtue of using Lighting Data Service.

<details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global">
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    
    <aura:attribute name="record" type="Object"/>
    <aura:attribute name="simpleRecord" type="Object"/>
    <aura:attribute name="recordError" type="String"/>
    
    <!-- config attributes -->
    <aura:attribute name="namefield" type="String" default="Name"/>
    <aura:attribute name="additionalfields" type="String"/>

    <!-- config attribute output helpers -->
    <aura:attribute name="namevalue" type="String"/>
    <aura:attribute name="additionalfielddata" type="Object[]"/>
    
    <!-- record that we're sitting on -->
    <aura:attribute name="recordfields" type="String[]" default="CreatedBy.Name,CreatedDate,OwnerId"/>
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.handleRecordUpdated}"
                      />
    
    <aura:attribute name="recordOwnerId" type="Id"/> <!-- assigned in Javascript after record load -->
    <aura:attribute name="recordOwner" type="Object"/>
    <aura:attribute name="simpleRecordOwner" type="Object"/>
    <aura:attribute name="recordErrorOwner" type="String"/>
    
    <!-- second instance of recordData component to get related record information -->
    <!-- owner of the record -->
    <force:recordData aura:id="recordOwner"
                      recordId="{!v.recordOwnerId}"
                      targetRecord="{!v.recordOwner}"
                      fields="Name,SmallPhotoUrl"
                      targetFields="{!v.simpleRecordOwner}"
                      targetError="{!v.recordErrorOwner}"
                      />
    
    <article class="slds-card">
        <div class="slds-card__header slds-grid">
            <header class="slds-media slds-media_center slds-has-flexi-truncate">
                <div class="slds-media__body">
                    <h2>
                        <span class="slds-text-heading_small">Record Info</span>
                    </h2>
                </div>
            </header>
        </div>
        <div class="slds-card__body slds-card__body_inner">
            <!-- Owner Picture -->
            <label>Owner</label><br /><lightning:avatar src="{!v.simpleRecordOwner.SmallPhotoUrl}"
                                                        fallbackIconName="utility:inbox" alternativeText="Salesforce"/>&nbsp;
            <!-- Owner Name --> <!-- what if owned by queue? --> <!-- what if owned by Master -->
            <ui:outputText value="{!v.simpleRecordOwner.Name}"/><br />
            
            
            <!-- Record Name -->
            <label>Name:</label>&nbsp;<ui:outputText value="{!v.namevalue}"/><br />
            <!-- CreatedDate, CreatedBy -->
            <label>Created By:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.CreatedBy.Name}"/>&nbsp;on
            <lightning:formattedDateTime value="{!v.simpleRecord.CreatedDate}" year="2-digit" month="short" day="2-digit" weekday="long"/>
            <br />
            
            <aura:if isTrue="{!v.additionalfielddata.length > 0}">
                <br /><span class="slds-text-title_caps">Additional Fields</span><br />
                <aura:iteration items="{!v.additionalfielddata}" var="field">
                    <label>{!field.name}:</label>&nbsp;{!field.value}<br />
                </aura:iteration>
            </aura:if>
            
            <!-- Display Lightning Data Service errors, if any -->
            <aura:if isTrue="{!not(empty(v.recordError))}">
                <div class="recordError">
                    <ui:message title="Error" severity="error" closable="true">
                        {!v.recordError}
                    </ui:message>
                </div>
            </aura:if>
            
            <!-- Display Lightning Data Service errors, if any -->
            <aura:if isTrue="{!not(empty(v.recordErrorOwner))}">
                <div class="recordError">
                    <ui:message title="Error" severity="error" closable="true">
                        {!v.recordErrorOwner}
                    </ui:message>
                </div>
            </aura:if>
        </div>
    </article>
</aura:component>
```

</p>
</details>


<details><summary>Full Controller</summary>
<p>

```html
({
    doInit: function(component, event, helper){
        // merge fields into the query field set
        // get the default list of fields from the component
        var recordFields = component.get('v.recordfields');
        // add the Configuration Attribute that the user set to the list of
        // fields we want to pull back
        recordFields.push(component.get('v.namefield'));
        
        // Get the list of other fields the user wants to display
        // this should be a comma separated list of field API names
        // add those to the list of fields to get back from the component
        var additionalFields = component.get('v.additionalfields');
        
        // if the user specified additional fields then split them on the comma
        // and add them to the list of fields to get from Lightning Data Service
        if(additionalFields != undefined){
            // create a list of
            var splitAdditionalFields = additionalFields.split(',');
            // smash two lists into each other
            recordFields = recordFields.concat(splitAdditionalFields);
        }
        // put he updated list of fields back onto the componenet
        component.set('v.recordfields',recordFields);
        
        // didn't set the record id until now so we could adjust the fields we're requesting
        var recordId = component.get('v.recordId');
        
        component.set('v.loaderId', recordId);
        // force the component to reload
        component.find('recordLoader').reloadRecord(true);
    },
    handleRecordUpdated: function(component, event, helper) {
        var eventParams = event.getParams();
        if(eventParams.changeType === "LOADED" || eventParams.changeType === "CHANGED") {
            // base record is loaded from Lighting Data Service

            // get the simpleRecord attribute from the component
            var simpleRecord = component.get('v.simpleRecord');
            
            // capture name value
            var nameField = component.get('v.namefield');
            component.set('v.namevalue',simpleRecord[nameField]);
            
            // get the list of additional fields from the 
            // Configuration Attribute
            var additionalFields = component.get('v.additionalfields');
            var splitAdditionalFields;
            // If the user supplied any data
            if(additionalFields != undefined){
                splitAdditionalFields = additionalFields.split(',');
            }
            
            // get the attribute to hold the additional field data 
            var additionalfielddata = []
            
            // loop through defined fields, build an object and
            // push it onto the stack to be displayed by the component
            if(splitAdditionalFields != undefined){
                for(var i = 0; i < splitAdditionalFields.length; i++){
                    var field = splitAdditionalFields[i].trim();
                    var value = simpleRecord[field];
                    additionalfielddata.push({name:field,value:value});
                }
            }
            // push the data back to the component
            component.set('v.additionalfielddata',additionalfielddata);
            
            var ownerId = component.get('v.recordOwnerId');
            // if this is the first time we're loading the record then 
            // populate the ownerid and load the record
            if(ownerId == null){
				
                // assign the base record's ownerid onto the attribute we've
                // attached to the second force:recordData's recordid value.
                component.set('v.recordOwnerId', simpleRecord.OwnerId);
                // fire the component event to reload the record now
                // that we've given it a record id.
                component.find('recordOwner').reloadRecord(true);
            }
            
        } else if(eventParams.changeType === "REMOVED") {
            // record is deleted
        } else if(eventParams.changeType === "ERROR") {
            // there’s an error while loading, saving, or deleting the record
        }
    }

})
```

</p>
</details>


## Phase 4

Finally, add some Apex to do things we can’t do without it, though we tried. So far we have accomplished a lot with zero Apex. It is a testament to the strength of the component framework that we can go this far and with this much flexibility.

### Get the “name” field for the object so we don’t have to define it
One of the annoying things to this point is that we have to ask the user which field to get the name from. We also don't have the correct field label for whatever the name field is. What if the user is viewing your record in a different language? Your labels should respect their chosen language.

Salesforce has all this information in the metadata, it just isn't something that our Lightning Component has access to. Because we're doing so much dynamically Salesforce doesn't get enough hints to know what metadata to surface and bring up to the client. I haven't figured out a way be this dynamic and still convince the framework to do all the heavy lifting here. (maybe dynamic component creation? You tell me.)

Our answer? Make our own server calls to get the metadata needed. We've got to make lots of changes. This will include changes to the Component to change its data source for the name field label, change the componenet controller to call apex when it starts and process the results when Lightning Data Service returns, moving some of our logic to the helper file out of the controller, and adding an Apex Controller.

To make this work we have to add and Apex controller with an @AuraEnabled method. From there will will teach our component controller to call the apex controller.

Using Developer Console select New -> Apex Class and name it RecordInfoController

Apex Controller
```java
public with sharing class RecordInfoController {

	// AuraEnabled makes this accessible to the Lighting Component
    @AuraEnabled
    // Lightning methods must be static to be accessible
    // We're returning a Map<String, Object> so we don't have to build a specific
    // class just to get data up to the parent, it does mean we have to be just a little
    // more creative in how we structure the data we're passing to it
    // by passing in the object name we're helping increase the cache hit rate down the road
    public static Map<String, Object> getRecordInfo(String sObjectName){
    	// Initialize the object to send it back later
        Map<String, Object> recordInfo = new Map<String, Object>();
        // pass the object name into the Schema methods to get the object details
        List<Schema.DescribeSobjectResult> results = Schema.describeSObjects(new List<String>{sObjectName});
        // retrieve the object label so we can display it on the component 
        // along with the api name, add it to the return object
        recordInfo.put('objectlabel',results[0].getLabel());
        // loop through all the fields on the object
        // look for one that Salesforce has tagged as isNameField, every object has one
        for(Schema.SObjectField fld : results[0].fields.getMap().Values()){
            Schema.DescribeFieldResult f = fld.getDescribe();
            if(f.isNameField()){
                Map<String,String> name = new Map<String,String>();
                name.put('apiname',f.getname());
                name.put('label',f.getLabel());
                recordInfo.put('name',name);
                break;
            }
        }
        // send the data back to the component
        return recordInfo;
    }
}
```
  
Changes to the Component

Change the componenet definition to attach to the Apex Controller
```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global"
                controller="RecordInfoController">
```
Add the attribute to hold the retun data from the Apex call
```html
    <aura:attribute name="recordinfo" type="Object"/>
```

Change the Name output to use the label from Apex
```html
<!-- Record Name -->
            <label>{!v.recordinfo.name.label}:</label>&nbsp;<ui:outputText value="{!v.namevalue}"/><br />
```

We're going to make dynamic changes to the component controller, we're going to rebuild the doInit function to just manage the apex callout, all the stuff we have in there right now will be moved out to the component helper.

To create the helper file we'll just click Helper in the Developer Console and Salesforce will get the file built and displayed. We'll move a bunch of the stuff that used to be in the controller out into the helper.

Helper file
```javascript
({
	buildcomponentdata : function(component, response) {
        
        // Alert the user with the value returned 
        // from the server
        var recordinfo = response.getReturnValue();
        component.set('v.recordinfo', response.getReturnValue());
        
        // add the API name of the name field from Apex to the component
        component.set('v.namefield', recordinfo.name.apiname);
        
		// merge fields into the query field set
        // get the default list of fields from the component
        var recordFields = component.get('v.recordfields');
		// add the Apex name field to the list of fields for Lightning Data Service
        recordFields.push(recordinfo.name.apiname);
        
        // Get the list of other fields the user wants to display
        // this should be a comma separated list of field API names
        // add those to the list of fields to get back from the component
        var additionalFields = component.get('v.additionalfields');
        
        // if the user specified additional fields then split them on the comma
        // and add them to the list of fields to get from Lightning Data Service
        if(additionalFields != undefined){
            // create a list of
            var splitAdditionalFields = additionalFields.split(',');
            // smash two lists into each other
            recordFields = recordFields.concat(splitAdditionalFields);
        }
        // put he updated list of fields back onto the componenet
        component.set('v.recordfields',recordFields);
        
        // didn't set the record id until now so we could adjust the fields we're requesting
        var recordId = component.get('v.recordId');
        
        component.set('v.loaderId', recordId);
        // force the component to reload
        component.find('recordLoader').reloadRecord(true);
	}
})
```

Change the entire doInit method in the controller to call the apex method, when that method returns call into the helper method to process the results. 
```javascript
    doInit: function(component, event, helper){
                
        //////////////////////////////////////
        var action = component.get("c.getRecordInfo");
        action.setParams({ sObjectName : component.get("v.sObjectName") });
		action.setStorable();
        // Create a callback that is executed after 
        // the server-side action returns
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {
                helper.buildcomponentdata(component, response);
            }
            else if (state === "INCOMPLETE") {
                // do something
            }
                else if (state === "ERROR") {
                    var errors = response.getError();
                    if (errors) {
                        if (errors[0] && errors[0].message) {
                            console.log("Error message: " + 
                                        errors[0].message);
                        }
                    } else {
                        console.log("Unknown error");
                    }
                }
        });
        $A.enqueueAction(action);
    },
```

With all of this in place you should be able to refresh the Account or Case page and see _Name_ replaced with Account Name or Case Number automatically. You could even go and change your language and see the label for the field change.

  <details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global"
                controller="RecordInfoController">
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    
    <aura:attribute name="record" type="Object"/>
    <aura:attribute name="simpleRecord" type="Object"/>
    <aura:attribute name="recordError" type="String"/>
    
    <!-- config attributes -->
    <aura:attribute name="namefield" type="String" default="Name"/>
    <aura:attribute name="additionalfields" type="String"/>

    <!-- config attribute output helpers -->
    <aura:attribute name="namevalue" type="String"/>
    <aura:attribute name="additionalfielddata" type="Object[]"/>
    
    <!-- data we got back from apex -->
    <aura:attribute name="recordinfo" type="Object"/>
    
    <!-- record that we're sitting on -->
    <aura:attribute name="recordfields" type="String[]" default="CreatedBy.Name,CreatedDate,OwnerId"/>
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.handleRecordUpdated}"
                      />
    
    <aura:attribute name="recordOwnerId" type="Id"/> <!-- assigned in Javascript after record load -->
    <aura:attribute name="recordOwner" type="Object"/>
    <aura:attribute name="simpleRecordOwner" type="Object"/>
    <aura:attribute name="recordErrorOwner" type="String"/>
    
    <!-- second instance of recordData component to get related record information -->
    <!-- owner of the record -->
    <force:recordData aura:id="recordOwner"
                      recordId="{!v.recordOwnerId}"
                      targetRecord="{!v.recordOwner}"
                      fields="Name,SmallPhotoUrl"
                      targetFields="{!v.simpleRecordOwner}"
                      targetError="{!v.recordErrorOwner}"
                      />
    
    <article class="slds-card">
        <div class="slds-card__header slds-grid">
            <header class="slds-media slds-media_center slds-has-flexi-truncate">
                <div class="slds-media__body">
                    <h2>
                        <span class="slds-text-heading_small">Record Info</span>
                    </h2>
                </div>
            </header>
        </div>
        <div class="slds-card__body slds-card__body_inner">
            <!-- Owner Picture -->
            <label>Owner</label><br /><lightning:avatar src="{!v.simpleRecordOwner.SmallPhotoUrl}"
                                                        fallbackIconName="utility:inbox" alternativeText="Salesforce"/>&nbsp;
            <!-- Owner Name --> <!-- what if owned by queue? --> <!-- what if owned by Master -->
            <ui:outputText value="{!v.simpleRecordOwner.Name}"/><br />
            
            
			<!-- Record Name -->
            <label>{!v.recordinfo.name.label}:</label>&nbsp;<ui:outputText value="{!v.namevalue}"/><br />
            <!-- CreatedDate, CreatedBy -->
            <label>Created By:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.CreatedBy.Name}"/>&nbsp;on
            <lightning:formattedDateTime value="{!v.simpleRecord.CreatedDate}" year="2-digit" month="short" day="2-digit" weekday="long"/>
            <br />
            
            <aura:if isTrue="{!v.additionalfielddata.length > 0}">
                <br /><span class="slds-text-title_caps">Additional Fields</span><br />
                <aura:iteration items="{!v.additionalfielddata}" var="field">
                    <label>{!field.name}:</label>&nbsp;{!field.value}<br />
                </aura:iteration>
            </aura:if>
            
            <!-- Display Lightning Data Service errors, if any -->
            <aura:if isTrue="{!not(empty(v.recordError))}">
                <div class="recordError">
                    <ui:message title="Error" severity="error" closable="true">
                        {!v.recordError}
                    </ui:message>
                </div>
            </aura:if>
            
            <!-- Display Lightning Data Service errors, if any -->
            <aura:if isTrue="{!not(empty(v.recordErrorOwner))}">
                <div class="recordError">
                    <ui:message title="Error" severity="error" closable="true">
                        {!v.recordErrorOwner}
                    </ui:message>
                </div>
            </aura:if>
        </div>
    </article>
</aura:component>
```

</p>
</details>

  <details><summary>Full Controller</summary>
<p>

```javascript
({
    doInit: function(component, event, helper){
                
        //////////////////////////////////////
        var action = component.get("c.getRecordInfo");
        action.setParams({ sObjectName : component.get("v.sObjectName") });
		action.setStorable();
        // Create a callback that is executed after 
        // the server-side action returns
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {
                helper.buildcomponentdata(component, response);
            }
            else if (state === "INCOMPLETE") {
                // do something
            }
                else if (state === "ERROR") {
                    var errors = response.getError();
                    if (errors) {
                        if (errors[0] && errors[0].message) {
                            console.log("Error message: " + 
                                        errors[0].message);
                        }
                    } else {
                        console.log("Unknown error");
                    }
                }
        });
        $A.enqueueAction(action);
    },
    handleRecordUpdated: function(component, event, helper) {
        var eventParams = event.getParams();
        if(eventParams.changeType === "LOADED" || eventParams.changeType === "CHANGED") {
            // base record is loaded from Lighting Data Service

            // get the simpleRecord attribute from the component
            var simpleRecord = component.get('v.simpleRecord');
            
            // capture name value
            var nameField = component.get('v.namefield');
            component.set('v.namevalue',simpleRecord[nameField]);
            
            // get the list of additional fields from the 
            // Configuration Attribute
            var additionalFields = component.get('v.additionalfields');
            var splitAdditionalFields;
            // If the user supplied any data
            if(additionalFields != undefined){
                splitAdditionalFields = additionalFields.split(',');
            }
            
            // get the attribute to hold the additional field data 
            var additionalfielddata = []
            
            // loop through defined fields, build an object and
            // push it onto the stack to be displayed by the component
            if(splitAdditionalFields != undefined){
                for(var i = 0; i < splitAdditionalFields.length; i++){
                    var field = splitAdditionalFields[i].trim();
                    var value = simpleRecord[field];
                    additionalfielddata.push({name:field,value:value});
                }
            }
            // push the data back to the component
            component.set('v.additionalfielddata',additionalfielddata);
            
            var ownerId = component.get('v.recordOwnerId');
            // if this is the first time we're loading the record then 
            // populate the ownerid and load the record
            if(ownerId == null){
				
                // assign the base record's ownerid onto the attribute we've
                // attached to the second force:recordData's recordid value.
                component.set('v.recordOwnerId', simpleRecord.OwnerId);
                // fire the component event to reload the record now
                // that we've given it a record id.
                component.find('recordOwner').reloadRecord(true);
            }
            
        } else if(eventParams.changeType === "REMOVED") {
            // record is deleted
        } else if(eventParams.changeType === "ERROR") {
            // there’s an error while loading, saving, or deleting the record
        }
    }

})
```
</p>
</details>

Design file remains unchanged, only because it is really difficult to delete an attribute.

Full Helper and Apex Controller can be found above.



## Phase 5

So we finally figured out how to give this component some real intelligence, go and figure out the name and the correct way to present it without asking the user for qualifying information.

Our final step builds on the Additional Fields concept. Instead of passing API names to the component we're going to build Custom Metadata to configure which fields to display for each SObject type. This will give us some of the same magic that we got out of the Name field, we can get the field labels instead of the API names, we can also centralize configuration which is a two-edged sword.

### Create a Custom Metadata Object

We need to go to Setup in Salesforce, using search find Custom Metadata Types, click New.

* Name = `Record Info Field`
* Plural Label `Record Info Fields`
* Object Name = `Record_Info_Field`
* Description = `Used to mark fields to be included in the Record Info lighting component.`


Once the object is created we want to add a few fields to it

Fields
* Object
	* Type = `Metadata Relationship`
	* Related To = `Entity Definition`
	* Label = `Object`
* Field
	* Type = `Metadata Relationship`
	* Related To = `Field Definition`
	* Label = 'Field'
	* Controlling Field = `Object`


Now that our configuration object is built, lets add some data. Select Manage Record Info Fields on the main page for the Record Info Field metadata page. Click New and create a few records for objects we're already working with; Account and Case.

I created records for `Account -> Employees` because I know that the API name and the label are different so this is a great test to make sure we get the field label correctly. You can add whatever your heart wants.

### Add Apex methods to the Lighting Component to customize the component per-object type with additional output fields

We've setup the system to allow us to manage this, now we again have to teach our component to handle this output.

We don't have to make any changes to the actual component as we're going to inject this into the current Additional Fields data output.

Apex Controller Changes, add this under the recordInfo variable definition
```java
        List<Map<String, String>> mdtfields = new List<Map<String, String>>();
        for(Record_Info_Field__mdt mdt : [select id, Field__r.QualifiedApiName, Field__r.label from Record_Info_Field__mdt where Object__r.QualifiedApiName = :sObjectName]){
            Map<String, String> riField = new Map<String, String>();
            riField.put('apiname', mdt.Field__r.QualifiedApiName);
            riField.put('label', mdt.Field__r.Label);
            mdtfields.add(riField);
        }
        recordInfo.put('mdtfields',mdtfields);
```
This will pull the metadata from the database and add it to the data structure we're returning up to the component.

Inside the helper's buildcomponentdata method lets enable processing of the new metadata
Under `recordFields.push(recordinfo.name.apiname);` add
```javascript
	// add the field API names from the custom metadata
    for(var i = 0; i < recordinfo.mdtfields.length; i++){
        recordFields.push(recordinfo.mdtfields[i].apiname);
    }
```

Now lets tell the controller that when Lighting Data Service loads the record it should also process the custom metadata.

Under `var additionalfielddata = [];` add
```javascript
  // loop through the custom metadata and add to Additional Fields section
  var recordinfo = component.get('v.recordinfo');
  if(recordinfo.mdtfields != undefined){
  	for(var i = 0; i < recordinfo.mdtfields.length; i++){
  		var field = recordinfo.mdtfields[i].label;
  		var value = simpleRecord[recordinfo.mdtfields[i].apiname];
  		additionalfielddata.push({name:field,value:value});
  	}
  }
```


Now you should be able to refresh your record and see the new fields displayed in the Additional Fields section.


## FINAL FILES

<details><summary>Full Component</summary>
<p>

```html
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId,force:hasSObjectName" access="global"
                controller="RecordInfoController">
    <aura:handler name="init" value="{!this}" action="{!c.doInit}"/>
    
    <aura:attribute name="record" type="Object"/>
    <aura:attribute name="simpleRecord" type="Object"/>
    <aura:attribute name="recordError" type="String"/>
    
    <!-- config attributes -->
    <aura:attribute name="namefield" type="String" default="Name"/>
    <aura:attribute name="additionalfields" type="String"/>

    <!-- config attribute output helpers -->
    <aura:attribute name="namevalue" type="String"/>
    <aura:attribute name="additionalfielddata" type="Object[]"/>
    
    <!-- data we got back from apex -->
    <aura:attribute name="recordinfo" type="Object"/>
    
    <!-- record that we're sitting on -->
    <aura:attribute name="recordfields" type="String[]" default="CreatedBy.Name,CreatedDate,OwnerId"/>
    <force:recordData aura:id="recordLoader"
                      recordId="{!v.recordId}"
                      targetRecord="{!v.record}"
                      fields="{!v.recordfields}"
                      targetFields="{!v.simpleRecord}"
                      targetError="{!v.recordError}"
                      recordUpdated="{!c.handleRecordUpdated}"
                      />
    
    <aura:attribute name="recordOwnerId" type="Id"/> <!-- assigned in Javascript after record load -->
    <aura:attribute name="recordOwner" type="Object"/>
    <aura:attribute name="simpleRecordOwner" type="Object"/>
    <aura:attribute name="recordErrorOwner" type="String"/>
    
    <!-- second instance of recordData component to get related record information -->
    <!-- owner of the record -->
    <force:recordData aura:id="recordOwner"
                      recordId="{!v.recordOwnerId}"
                      targetRecord="{!v.recordOwner}"
                      fields="Name,SmallPhotoUrl"
                      targetFields="{!v.simpleRecordOwner}"
                      targetError="{!v.recordErrorOwner}"
                      />
    
    <article class="slds-card">
        <div class="slds-card__header slds-grid">
            <header class="slds-media slds-media_center slds-has-flexi-truncate">
                <div class="slds-media__body">
                    <h2>
                        <span class="slds-text-heading_small">Record Info</span>
                    </h2>
                </div>
            </header>
        </div>
        <div class="slds-card__body slds-card__body_inner">
            <!-- Owner Picture -->
            <label>Owner</label><br /><lightning:avatar src="{!v.simpleRecordOwner.SmallPhotoUrl}"
                                                        fallbackIconName="utility:inbox" alternativeText="Salesforce"/>&nbsp;
            <!-- Owner Name --> <!-- what if owned by queue? --> <!-- what if owned by Master -->
            <ui:outputText value="{!v.simpleRecordOwner.Name}"/><br />
            
            
			<!-- Record Name -->
            <label>{!v.recordinfo.name.label}:</label>&nbsp;<ui:outputText value="{!v.namevalue}"/><br />
            <!-- CreatedDate, CreatedBy -->
            <label>Created By:</label>&nbsp;<ui:outputText value="{!v.simpleRecord.CreatedBy.Name}"/>&nbsp;on
            <lightning:formattedDateTime value="{!v.simpleRecord.CreatedDate}" year="2-digit" month="short" day="2-digit" weekday="long"/>
            <br />
            
            <aura:if isTrue="{!v.additionalfielddata.length > 0}">
                <br /><span class="slds-text-title_caps">Additional Fields</span><br />
                <aura:iteration items="{!v.additionalfielddata}" var="field">
                    <label>{!field.name}:</label>&nbsp;{!field.value}<br />
                </aura:iteration>
            </aura:if>
            
            <!-- Display Lightning Data Service errors, if any -->
            <aura:if isTrue="{!not(empty(v.recordError))}">
                <div class="recordError">
                    <ui:message title="Error" severity="error" closable="true">
                        {!v.recordError}
                    </ui:message>
                </div>
            </aura:if>
            
            <!-- Display Lightning Data Service errors, if any -->
            <aura:if isTrue="{!not(empty(v.recordErrorOwner))}">
                <div class="recordError">
                    <ui:message title="Error" severity="error" closable="true">
                        {!v.recordErrorOwner}
                    </ui:message>
                </div>
            </aura:if>
        </div>
    </article>
</aura:component>
```

</p>
</details>

<details><summary>Full Controller</summary>
<p>

```javascript
({
    doInit: function(component, event, helper){
                
        //////////////////////////////////////
        var action = component.get("c.getRecordInfo");
        action.setParams({ sObjectName : component.get("v.sObjectName") });
		action.setStorable();
        // Create a callback that is executed after 
        // the server-side action returns
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {
                helper.buildcomponentdata(component, response);
            }
            else if (state === "INCOMPLETE") {
                // do something
            }
                else if (state === "ERROR") {
                    var errors = response.getError();
                    if (errors) {
                        if (errors[0] && errors[0].message) {
                            console.log("Error message: " + 
                                        errors[0].message);
                        }
                    } else {
                        console.log("Unknown error");
                    }
                }
        });
        $A.enqueueAction(action);
    },
    handleRecordUpdated: function(component, event, helper) {
        var eventParams = event.getParams();
        if(eventParams.changeType === "LOADED" || eventParams.changeType === "CHANGED") {
            // base record is loaded from Lighting Data Service

            // get the simpleRecord attribute from the component
            var simpleRecord = component.get('v.simpleRecord');
            
            // capture name value
            var nameField = component.get('v.namefield');
            component.set('v.namevalue',simpleRecord[nameField]);
            
            // get the list of additional fields from the 
            // Configuration Attribute
            var additionalFields = component.get('v.additionalfields');
            var splitAdditionalFields;
            // If the user supplied any data
            if(additionalFields != undefined){
                splitAdditionalFields = additionalFields.split(',');
            }
            
            // get the attribute to hold the additional field data 
            var additionalfielddata = [];
            
            // loop through the custom metadata and add to Additional Fields section
            var recordinfo = component.get('v.recordinfo');
            if(recordinfo != undefined && recordinfo.mdtfields != undefined){
                for(var i = 0; i < recordinfo.mdtfields.length; i++){
                    var field = recordinfo.mdtfields[i].label;
                    var value = simpleRecord[recordinfo.mdtfields[i].apiname];
                    additionalfielddata.push({name:field,value:value});
                }
            }
            
            // loop through defined fields, build an object and
            // push it onto the stack to be displayed by the component
            if(splitAdditionalFields != undefined){
                for(var i = 0; i < splitAdditionalFields.length; i++){
                    var field = splitAdditionalFields[i].trim();
                    var value = simpleRecord[field];
                    additionalfielddata.push({name:field,value:value});
                }
            }
            // push the data back to the component
            component.set('v.additionalfielddata',additionalfielddata);
            
            var ownerId = component.get('v.recordOwnerId');
            // if this is the first time we're loading the record then 
            // populate the ownerid and load the record
            if(ownerId == null){
				
                // assign the base record's ownerid onto the attribute we've
                // attached to the second force:recordData's recordid value.
                component.set('v.recordOwnerId', simpleRecord.OwnerId);
                // fire the component event to reload the record now
                // that we've given it a record id.
                component.find('recordOwner').reloadRecord(true);
            }
            
        } else if(eventParams.changeType === "REMOVED") {
            // record is deleted
        } else if(eventParams.changeType === "ERROR") {
            // there’s an error while loading, saving, or deleting the record
        }
    }

})
```

</p>
</details>

<details><summary>Full Helper</summary>
<p>

```javascript
({
	buildcomponentdata : function(component, response) {
        
        // Alert the user with the value returned 
        // from the server
        var recordinfo = response.getReturnValue();
        component.set('v.recordinfo', recordinfo);
        
        // add the API name of the name field from Apex to the component
        component.set('v.namefield', recordinfo.name.apiname);
        
		// merge fields into the query field set
        // get the default list of fields from the component
        var recordFields = component.get('v.recordfields');
		// add the Apex name field to the list of fields for Lightning Data Service
        recordFields.push(recordinfo.name.apiname);
        // add the field API names from the custom metadata
        for(var i = 0; i < recordinfo.mdtfields.length; i++){
            recordFields.push(recordinfo.mdtfields[i].apiname);
        }
        
        // Get the list of other fields the user wants to display
        // this should be a comma separated list of field API names
        // add those to the list of fields to get back from the component
        var additionalFields = component.get('v.additionalfields');
        
        // if the user specified additional fields then split them on the comma
        // and add them to the list of fields to get from Lightning Data Service
        if(additionalFields != undefined){
            // create a list of
            var splitAdditionalFields = additionalFields.split(',');
            // smash two lists into each other
            recordFields = recordFields.concat(splitAdditionalFields);
        }
        // put he updated list of fields back onto the componenet
        component.set('v.recordfields',recordFields);
        
        // didn't set the record id until now so we could adjust the fields we're requesting
        var recordId = component.get('v.recordId');
        
        component.set('v.loaderId', recordId);
        // force the component to reload
        component.find('recordLoader').reloadRecord(true);
	}
})
```

</p>
</details>

<details><summary>Full Design</summary>
<p>

```html
<design:component label="Record Info2">
    <design:attribute name="namefield" label="Name Field" />
    <design:attribute name="additionalfields" label="Additional Fields" />
</design:component>
```

</p>
</details>

<details><summary>Full Style</summary>
<p>

```css
.THIS label {
    font-weight: bold;
}
```

</p>
</details>

<details><summary>Full Apex Controller</summary>
<p>

```java
public with sharing class RecordInfoController {

	// AuraEnabled makes this accessible to the Lighting Component
    // Lightning methods must be static to be accessible
    // We're returning a Map<String, Object> so we don't have to build a specific
    // class just to get data up to the parent, it does mean we have to be just a little
    // more creative in how we structure the data we're passing to it
    // by passing in the object name we're helping increase the cache hit rate down the road
    @AuraEnabled
    public static Map<String, Object> getRecordInfo(String sObjectName){
    	// Initialize the object to send it back later
        Map<String, Object> recordInfo = new Map<String, Object>();
        
        // build the mdtfields list and populate it based on what is in the database
        List<Map<String, String>> mdtfields = new List<Map<String, String>>();
        for(Record_Info_Field__mdt mdt : [select id, Field__r.QualifiedApiName, Field__r.label from Record_Info_Field__mdt where Object__r.QualifiedApiName = :sObjectName]){
            Map<String, String> riField = new Map<String, String>();
            riField.put('apiname', mdt.Field__r.QualifiedApiName);
            riField.put('label', mdt.Field__r.Label);
            mdtfields.add(riField);
        }
        recordInfo.put('mdtfields',mdtfields);
        
        // pass the object name into the Schema methods to get the object details
        List<Schema.DescribeSobjectResult> results = Schema.describeSObjects(new List<String>{sObjectName});
        // retrieve the object label so we can display it on the component 
        // along with the api name, add it to the return object
        recordInfo.put('objectlabel',results[0].getLabel());
        // loop through all the fields on the object
        // look for one that Salesforce has tagged as isNameField, every object has one
        for(Schema.SObjectField fld : results[0].fields.getMap().Values()){
            Schema.DescribeFieldResult f = fld.getDescribe();
            if(f.isNameField()){
                Map<String,String> name = new Map<String,String>();
                name.put('apiname',f.getname());
                name.put('label',f.getLabel());
                recordInfo.put('name',name);
                break;
            }
        }
        // send the data back to the component
        return recordInfo;
    }
}
```

</p>
</details>

## Wrap up

Where can you see that we've cut corners?

Do you see any significant design changes that you would make?

How can you make the entire thing translated?

What if the record doesn't have an owner, or has multiple owners?

If you were to debug through this you would see that `handleRecordUpdated` is actually called twice, once using the default fields and again after we add our fields to it. how could you limit that?

What other data could be useful in this component? Would you add something like this to your org?

Feel free to chat with me @AKHeber on Twitter or in the issues on this repo.

