+++
date = "2016-07-29T16:18:23+05:30"
draft = true
title = "Build a Suggestion Box App with Lightning Experience"

+++


1. [Introduction](#introduction)
2. [Create a Server-side Apex Controller Class](#create-a-server-side-apex-controller-class)
3. [Create the SearchBar component](#create-the-searchbar-component)
4. [Create the SearchKey Event and SuggestionList Component](#create-the-searchkey-event-and-suggestionlist-component)
5. [Create a Suggestion Detail Component](#create-a-suggestion-detail-component)
6. [Summary](#summary)

## Introduction
In this project, you learn how to build a Lightning Application on App Cloud from start to finish.If you're new to App Cloud, the goal is to introduce you to the basics of app building together with introduction to the new lightning platform and basics to develop application using lightning components.If you're familiar with the App Cloud Admin features—managing users and security, customizing standard objects, and so on—the goal is to apply those skills to developing new application and learn how to extend the functionality of these applications using lightning. You need a Developer Edition org to complete this project. If you don't have one, you can sign up [here](https://developer.salesforce.com/signup).

You will build a Suggestion Box Lightning Application that allows employees to submit suggestions, search for existing suggestions and vote for them.All of this with the following steps:

* Install the pre-created building blocks of the Suggestion Box App by clicking [here](http://bit.ly/df16_sb_package)
  This package includes the app definition, data model, validation rules, process and reports and dashboard which together form the basic Suggestion box Application
* Extend this application using Lightning components to design a stand-alone Suggestion Box Application which will look somethng like this:
 
![alt text](https://github.com/sonamraju14/df16workshops/blob/master/content/workshop/suggestion-box/images/SB_app.png)

Each rectangle in this image represents a lightning component:
* Red box: Displays the "add suggestion" functionality
* Yellow box: Displays the "search suggestion" functionality
* Green box: Displays the "results" of the search functionality
* Blue box: Displays the "details of the selected record" in the search functionality
* Purple box: "encampasses all the above functionalities into a single component" which then sits inside a Lightning Application

Let's begin with exploring our prebuilt Suggestion Box App which was installed using the package.

All eyes on the screen!

Now, that we know and understand how we can build a basic application using point-and-click, let us now extend this app using lightning components.

The Lightning Component framework is a UI framework for developing dynamic web apps for mobile and desktop devices. In this workshop, you'll create a simple Lightning Application that is built of smaller components which will help you create, search and vote for existing suggestions in your org. You'll start by creating an Apex controller class, then create the Lightning Components and an event handler and finally render the Application UI using all the components together.

## Create a Server-side Apex Controller Class

Create a class to access data from Suggestion custom object:

1. In your DE environment, click Your **Name | Developer Console**
2. Select **File | New | Apex Class**
3. For the class name, enter **SuggestionController** and then click **OK**
4. In the body of the class (i.e. between the {} braces), enter the following code

```
public class SuggestionController {

  	@AuraEnabled
	public static List<Suggestion__c> getSuggestions() {
		return [SELECT id, Name, Status__c, Suggestion_Category__c, Suggestion_Description__c,	Implemented_Date__c, createdDate 
			FROM Suggestion__c 
			ORDER BY createdDate DESC];
	  }
    
    @AuraEnabled
    public static Suggestion__c saveSuggestion(Suggestion__c suggestion) {
        upsert suggestion;
        return suggestion;
    }
    
    @AuraEnabled
    public static List<Suggestion__c> findAll() {
        return [SELECT id, name, Suggestion_Description__c, Vote_up__c FROM Suggestion__c LIMIT 50];
    }
    
    @AuraEnabled
    public static List<Suggestion__c> findByName(String searchKey) {
        String name = '%' + searchKey + '%';
        return [SELECT id, name, Suggestion_Description__c, Vote_up__c FROM Suggestion__c WHERE name LIKE :name LIMIT 50];
    }  
   
    @AuraEnabled
    public static Suggestion__c findById(String suggestionId) {
        return [SELECT id, name, Status__c, Suggestion_Category__c, Suggestion_Description__c,Vote_up__c
                    FROM Suggestion__c WHERE Id = :suggestionId];
    }
    
    @AuraEnabled
    public static Suggestion__c voteSuggestion(String suggestionId) {
        Suggestion__c s = new Suggestion__c();
        s=[SELECT id, name, Status__c, Suggestion_Category__c, Suggestion_Description__c,Vote_up__c
                    FROM Suggestion__c WHERE Id =:suggestionId];
        
        s.Vote_up__c = s.Vote_up__c + 1;
        upsert s;
        return s;
    } 
} 
```
@AuraEnabled enables client and server-side access to the controller method. Select **File | Save**.


### Create the SuggestionBoxCreate Component

A lightning component is a combination of markup, JavaScript, and CSS. You first create a component bundle.

1. In the **Developer Console**, select **File | New | Lightning Component**
2. For the component name, enter **SuggestionBoxCreate** and then click **Submit**
3. Edit the aura:component tag, and specify the controller to use.Edit the code as shown below:

```
<aura:component controller="SuggestionController" implements="flexipage:availableForAllPageTypes">
   <ltng:require styles="{!$Resource.slds + 'assets/styles/salesforce-lightning-design-system-vf.css'}" />
   <aura:attribute name="suggestions" type="Suggestion__c[]" />
   <aura:attribute name="newSuggestion" type="Suggestion__c"
      default="{ 'sobjectType': 'Suggestion__c',
      'Name': '',
      'Status__c': '',
      'Suggestion_Category__c': '',
      'Suggestion_Description__c': ''
      }"></aura:attribute>
   <div class="container">
      <h1>
         Add Suggestions
         <div style="margin-left: 0; height: 30px; float: right; margin-top: -3px; margin-right: 0; auto; vertical-align: inherit;">
            <ui:button aura:id="addbutton" label="New" labelClass="assistiveText" class="myButton" press="{!c.addNew}" />
         </div>
      </h1>
   </div>
   <!-- Input Form using components -->
   <div aura:id="formbox" class="myboxhidden">
      <form>
         <fieldset>
            <ui:inputText aura:id="sugname" label="Suggestion Name"
               class="form-control"
               value="{!v.newSuggestion.Name}"
               placeholder="Suggestion Name" required="true"/>
            <ui:inputSelect aura:id="category" label="Suggestion Category"
               class="cExpenseForm form-control"
               value="{!v.newSuggestion.Suggestion_Category__c}"
               required="true" >
               <ui:inputSelectOption text="Customer Service" value="Customer Service"/>
               <ui:inputSelectOption text="Employee Services" value="Employee Services"/>
               <ui:inputSelectOption text="Facilities/ IT" value="Facilities/ IT" />
               <ui:inputSelectOption text="Kitchen Snacks" value="Implemented"/>
               <ui:inputSelectOption text="Others" value="Implemented"/>
            </ui:inputSelect>
            <ui:inputText aura:id="description" label="Suggestion Description"
               class="cExpenseForm form-control"
               value="{!v.newSuggestion.Suggestion_Description__c}"
               placeholder="Describe your suggestion here" />
            <ui:button label="Submit" press="{!c.createSuggestion}" />
         </fieldset>
      </form>
   </div>
</aura:component>
```
4. Select **File | Save**
5. In the button panel on the right, click **Controller**
6. In place of the myAction JavaScript function, add the following code:

```
({
   addNew: function(component, event, helper) 
    			{
                var el = component.find('formbox');
                if ($A.util.hasClass(el.getElement(), 'myboxhidden')) 
                {
                helper.showInput(component);
				} 
                    else {
				helper.hideInput(component);
						 }
				},
                
    createSuggestion: function(component, event, helper) 
    			{
                var newSuggestion = component.get("v.newSuggestion");
                helper.createSuggestion(component, newSuggestion);
				}
})
```
7. Select **File | Save**
8. In the button panel on the right, click **Helper**
9. In place of the helpermethod JavaScript function, add the following code:

```
({
    showInput: function(component) {
        var el = component.find('formbox');
        $A.util.removeClass(el.getElement(), 'myboxhidden');
        $A.util.addClass(el.getElement(), 'mybox');
    },

    hideInput: function(component) {
        var el = component.find('formbox');
        $A.util.addClass(el.getElement(), 'myboxhidden');
    },

    createSuggestion: function(component, suggestion) {
        this.upsertSuggestion(component, suggestion, function(a) {
            var suggestions = component.get("v.suggestions");
            suggestions.unshift(a.getReturnValue());
            component.set("v.suggestions", suggestions);
            this.hideInput(component);
        });
    },
    upsertSuggestion: function(component, suggestion, callback) {
        var action = component.get("c.saveSuggestion");
        action.setParams({
            "suggestion": suggestion
        });
        if (callback) {
            action.setCallback(this, callback);
        }
        $A.enqueueAction(action);
    }

})
```
10. Select **File | Save**
11. In the button panel on the right, click **Style**
12. In place of .THIS {}, add the following code:

```
.THIS h3 {
	margin: 0px;
}
.THIS h1 {
	font-size: 24pt;
	margin-top: 5px;
	margin-bottom: 5px;
	margin-left: 5px;
	margin-right: 5px;
    height: 35px;
    
}

.THIS.container {
	margin: 5px;
	border: 1px solid black;
	border-radius: 1px;
	background-color: rgb(200, 198, 198);
    font-size: 18pt;
}

.THIS .form-control {
    display: block;
    width: 100%;
    height: 34px;
    padding: 6px 14px;
    font-size: 14px;
    line-height: 1.42857143;
    color: #000000;
    background-color: #ffffff;
    background-image: none;
    border: 1px solid #cfd0d2;
    border-radius: 4px;
    -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
    box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
    -webkit-transition: border-color ease-in-out .15s, box-shadow ease-in-out .15s;
    -o-transition: border-color ease-in-out .15s, box-shadow ease-in-out .15s;
    transition: border-color ease-in-out .15s, box-shadow ease-in-out .15s;
}

.THIS .myButton {
	width: 35px;
	background-image: url(/resource/plusbutton);
	background-repeat: no-repeat;
	height: 35px;
	background-size: contain;
	/*margin-left: -50px;*/
	border-style: none;
	background-color: transparent;
}

.THIS .img {
	background: url(/resource/plusbutton/) no-repeat;
	width:50px;
	height:25px;
}

.THIS.mybox {
  /*display: block;*/
  opacity: 1;
  margin-left: 5px;
  margin-bottom: 5px;
  margin-right: 5px;
  transform: scale(1, 1);
  transition-property: transform, height;
  transition-duration: 1s, 1s;
}

.THIS.myboxhidden {
  /*display: none;*/
  transform: scale(0,0);
  height: 0;
  /*opacity: 0;*/
  transition-property: transform, height;
  transition-duration: 1s, 1.2s;
}
```

#### Code highlights:
* Lightning components can include regular HTML markup and other Lightning components
* The Server-side Apex Controller has methods which will be used by components to access and modify records in the database
* The controller in the Component bundle has javascript methods which use the component attributes and invoke server side controller method to process data. *CreateSuggestion* is one such method which invokes *savesuggestion* method in the *SuggestionController* apex class
*  *.THIS* in the CSS symbolises that the css written in the component bundle only applies to this specific component UI


## Create the SearchBar component
To be used by employees to search for existing suggestions

TODO

## Create the SearchKey Event and SuggestionList Component
We will create a suggestionlist component to list down the existing components in the system and an event to connect the searchbar and suggestionlist component

TODO

## Create a Suggestion Detail Component
We will create this component to display the suggestion selected by emeployee from the SuggestionList component.This component will have a vote up button so that employees can vote for the suggestions they like.

TODO

# Summary

TODO



