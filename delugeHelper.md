# Single Lookup Update :

```javascript
 	sinContactID = "1136779000000569290";
 	accID = "1136779000000569096";

 	sinCon = zoho.crm.getRecordById("Contacts", sinContactID);
// 	info sinCon;

 	sinMap = Map();
 	sinMap.put("Lookup",sinContactID);
 //	sinMap.put("Lookup",{"id":sinContactID});
 	info sinMap;

 	updateSinCon = zoho.crm.updateRecord("Accounts", accID, sinMap);
 	info updateSinCon;
```

Json Format -- for Updating in Single Lookpup --


```json
{"Lookup":"1136779000000569290"}
```


---

# Create a lead with status "Fresh" :


```javascript
lead = Map();

lead.put("Company","Test Company");
lead.put("First_Name","Test");
lead.put("Last_Name","Testing");
lead.put("Email","test@email.com");
lead.put("Phone","1234567890");
lead.put("Lead_Status","Fresh");

res = zoho.crm.createRecord("Leads", lead);
info res;


```


---

# Reminder for Email and Popup :


```javascript
taskMap = Map();

taskMap.put("Subject","New task is created in lead");
taskMap.put("Description","15 minute automated reminder");
taskMap.put("Priority","High");
taskMap.put("Status","Not Started");
taskMap.put("Due_Date",zoho.currentdate);

taskMap.put("What_Id",leadId);
taskMap.put("$se_module","Leads");

rem = zoho.currenttime.addMinutes(15).toString("yyyy-MM-dd'T'HH:mm:ss+05:30");

taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + rem});

info zoho.crm.createRecord("Tasks",taskMap);


```


---

# Create Quotes and Show in Deals :

```javascript
Note - Take Products(Related List) from Deal Record & create quotation with Quoted Items(Same from Related List from Products).

    dealId = "1136779000000569369";
    getProductData = zoho.crm.getRelatedRecords("Products", "Deals", dealId);
 
    info getProductData;
 
    productList = List();
    for each product in getProductData
    {
        productMap = Map();
        productMap.put("product", {"id": product.get("id")});
        productMap.put("quantity", 1);
//      productMap.put("list_price", product.get("Unit_Price").toDecimal());
        productList.add(productMap);
    }
 	info productList;
 
    quoteMap = Map();
    quoteMap.put("Subject", "Quote for Deal " + dealId);
    quoteMap.put("Deal_Name", dealId);
    quoteMap.put("Product_Details", productList);
 
    info quoteMap;
 
    res = zoho.crm.createRecord("Quotes", quoteMap);
    info res;
```

Json Format -- Create Quotes and Show in Deals --


```json
{
  "Subject": "Quote for Deal 1136779000000569369",
  "Deal_Name": "1136779000000569369",
  "Product_Details": [
    {
      "product": {
        "id": "1136779000000734026"
      },
      "quantity": 1
    },
    {
      "product": {
        "id": "1136779000000734021"
      },
      "quantity": 1
    },
    {
      "product": {
        "id": "1136779000000734016"
      },
      "quantity": 1
    },
    {
      "product": {
        "id": "1136779000000734011"
      },
      "quantity": 1
    }
  ]
}
```


---

# Add in Multi-Select Lookup in AccountsXContacts using invoke :


```javascript
accID = "1136779000000569096";
multContID1 = "1136779000000569290";
multContID2 = "1136779000000572469";
multContID3 = "1136779000000569287";

// response = invokeurl
// [
// 	url :"https://www.zohoapis.in/crm/v2.1/Accounts/" + accID
// 	type :GET
// 	connection:"getdata"
// ];
// info response;

contList = List();
cont1 = Map();
cont1.put("id",multContID1);
cont11 = Map();
cont11.put("Add_Contacts_1",cont1);
cont111 = Map();
cont111.put(cont11);
contList.add(cont111);
// info cont111;

cont2 = Map();
cont2.put("id",multContID2);
cont22 = Map();
cont22.put("Add_Contacts_1",cont2);
cont222 = Map();
cont222.put(cont22);
contList.add(cont222);
// info contList;

obj1Map = Map();
obj1Map.put("id",accID);
obj1Map.put("Add_Contacts",contList);
// info obj1Map;

objList = List();
objList.add(obj1Map);
multMap = Map();
multMap.put("data",objList);
info multMap;

resp = invokeurl
[
 	url :"https://www.zohoapis.in/crm/v8/Accounts/" + accID
 	type :PUT
 	parameters:multMap.toString()
 	connection:"getdata"
];

info resp;
```

Json Format -- Create Quotes and Show in Deals --


```json
{
  "data": [
    {
      "id": "1136779000000569096",
      "Add_Contacts": [
        {
          "Add_Contacts_1": {
            "id": "1136779000000569290"
          }
        },
        {
          "Add_Contacts_1": {
            "id": "1136779000000572469"
          }
        }
      ]
    }
  ]
}
```


---

# Find Duplicate in Leads and Contacts Module and Add in Multi-Pickup List ( when lead is created ) :


```javascript
leadId = "1147162000000641141";
getLeadData = zoho.crm.getRecordById("Leads",leadId);
leadMobile = getLeadData.getJson("Mobile");
leadPhone = getLeadData.getJson("Phone");
duplicateName = "";
// Check Lead Duplicate...
searchLead = zoho.crm.searchRecords("Leads", 
   "(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");	
	
leadSize = searchLead.size();

if(leadSize > 1){
	duplicateName ="Lead Duplicate";
	info duplicateName;
}else{
	duplicateName = "No Duplicate";
	info duplicateName;
}

// Check Contact Duplicate...

searchContact = zoho.crm.searchRecords("Contacts", 
   "(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");	

   if(searchContact.isEmpty()){
		info duplicateName;	
}else{

	if(duplicateName == "Lead Duplicate"){
		duplicateName = "Lead & Contact Duplicate";
		info duplicateName;
	}else{
		duplicateName = "Contact Duplicate";
		info duplicateName;
	}
	
}

if(duplicateName == "Lead Duplicate"){
	update = zoho.crm.updateRecord("Leads", leadId, {"Duplicate":duplicateName});
	info update;
}
else if(duplicateName == "Contact Duplicate"){
	update = zoho.crm.updateRecord("Leads", leadId, {"Duplicate":duplicateName});
	info update;
}
else if(duplicateName == "Lead & Contact Duplicate"){
	update = zoho.crm.updateRecord("Leads", leadId, {"Duplicate":duplicateName});
	info update;
}
```

Json Format -- for Adding in Multi-Pickup List --


```json
{"Duplicate":duplicateName}
```


---

# Add in Multi-Select Lookup in AccountsXContacts:

```javascript
 	accId = "1136779000000726001";
 	cont1 = "1136779000000569289";
 	cont2 = "1136779000000569290";
 
 	contList = List();
 	contList.add(cont1);
 	contList.add(cont2);
 
 	res=zoho.crm.getRecords("Accounts_X_Contacts");
 	info res;

 	for each cont in contList
 	{
 		mp=map();
 		mp.put("add_contacts",accId);
 		mp.put("Add_Contacts_1",cont);
 		create=zoho.crm.createRecord("Accounts_X_Contacts", mp);
 		info create;
 	}
```


Json Format -- for Updating in Multi - Lookpup in AccountsXContacts --


```json
{"add_contacts":"1136779000000726001", // accId
"Add_Contacts_1":"1136779000000569290"} // lookupId from getRecords linking module
```


---

# Auto Folder Creation In WorkDrive For Deals :

```javascript
dealId = "1234567890";
folderId = "jw0g37ef3b4f25b514cecabb2c5c16d785023";
header = Map();
header.put("Accept","application/vnd.api+json");
attMap = Map();
attMap.put("name",dealId);
attMap.put("parent_id",folderId);
// info attMap;

wrapMap = Map();
wrapMap.put("attributes",attMap);
wrapMap.put("type","files");
// info wrapMap;

dataMap = Map();
dataMap.put("data",wrapMap);
// info dataMap;

response = invokeurl
[
 	url :" https://www.zohoapis.in/workdrive/api/v1/files"
 	type :POST
 	parameters:dataMap.toString()
 	headers:header
 	connection:"workdrive"
];

info response;
newFolderId = response.get("data").get("attributes").get("permalink");
// info newFolderId;

urlMap = Map();
urlMap.put("URL",newFolderId);
// 	info urlMap;

updateDeal = zoho.crm.updateRecord("Deals",dealId,urlMap);
info updateDeal;
```

Json Format -- for autoFolderCreationInWorkDriveForDeals --


```json
{
  "data": {
    "attributes": {
      "name": 1234567890,
      "parent_id": "jw0g37ef3b4f25b514cecabb2c5c16d785023"
    },
    "type": "files"
  }
}
```


---

# Add in Multi-Select Lookup in AccountsXUsers:

```javascript
 userId = "1136779000000426001";

 // info zoho.crm.getRecords("Accounts_X_Users");

 mp=map();
 mp.put("userlookup221_3",accID);
 mp.put("User",userId);
 create=zoho.crm.createRecord("Accounts_X_Users", mp);
 info create;

```

Json Format -- for Updating in Multi - Lookpup in AccountsXContacts --


```json
{"userlookup221_3":"1136779000000569096", // accId
"User":"1136779000000426001"}
```


---

# Upload files from Attachments in Deal Record to WorkDrive :


```javascript
// 	dealId = "1136779000000758081";
workdriveFolderId = "jw0g37ef3b4f25b514cecabb2c5c16d785023";
fileName = "delugeRemainder.txt";
header = Map();
header.put("Accept","application/vnd.api+json");
getFileData = zoho.crm.getRelatedRecords("Attachments","Deals",dealId);
//     info res;
for each  data in getFileData
{
 	file = invokeurl
 	[
 		url :data.get("$link_url")
 		type :GET
 	];
 	// 	info file;
 	list_of_text = List();
 	list_of_text.add({"paramName":"filename","content":data.get("File_Name"),"stringPart":"true"});
 	list_of_text.add({"paramName":"parent_id","content":workdriveFolderId,"stringPart":"true"});
 	list_of_text.add({"paramName":"override-name-exist","content":"true","stringPart":"true"});
 	list_of_text.add({"paramName":"content","content":file,"stringPart":"false"});
 	// 	info list_of_text;
 	response = invokeurl
 	[
 		url :"https://www.zohoapis.in/workdrive/api/v1/upload"
 		type :POST
 		headers:header
 		files:list_of_text
 		connection:"workdrive"
 	];
 	info response.get("data").getJson("attributes").get("Permalink");
}

return "";
```

Json Format -- for Uplad File In WorkDrive From Attachments in Deals --


```json
[
  {
    "paramName": "filename",
    "content": "delugeRemainder.txt",
    "stringPart": "true"
  },
  {
    "paramName": "parent_id",
    "content": "jw0g37ef3b4f25b514cecabb2c5c16d785023",
    "stringPart": "true"
  },
  {
    "paramName": "override-name-exist",
    "content": "true",
    "stringPart": "true"
  },
  {
    "paramName": "content",
    "content": "<!DOCTYPE html></body></html>",
    "stringPart": "false"
  }
]
```


---

# Update Tag -- It add new tag & remove old tag : 


```javascript
	dealId = "1136779000000758081";
	getDealData = zoho.crm.getRecordById("Deals", dealId);
	existingTags = getDealData.get("Tag");
	// info existingTags;
	
	existingTags = List();
	
	newTag1 = Map();
	newTag1.put("name","POP");
	existingTags.add(newTag1);
	
	newTag2 = Map();
	newTag2.put("name","POPPY");
	existingTags.add(newTag2);
	
	updateMap = Map();
	updateMap.put("Tag",existingTags);
	info updateMap;
	update = zoho.crm.updateRecord("Deals", dealId, updateMap);
	info update;
```

Json Format -- Update Tag --


```json
{
  "Tag": [
    {
      "name": "POP"
    },
    {
      "name": "POPPY"
    }
  ]
}
```


---

# Add a new Tag and keep the existing one's :

```javascript
dealId = "1136779000000758081";
	getDealData = zoho.crm.getRecordById("Deals", dealId);
	dealTag = getDealData.get("Tag");
	if(dealTag == null)
	{
	  dealTag = List();
	}

	newTag = Map();
	newTag.put("name","POP");
	dealTag.add(newTag);
	
	newTag1 = Map();
	newTag1.put("name","POPPY");
	dealTag.add(newTag1);
	
	updateMap = Map();
	updateMap.put("Tag",dealTag);
// info updateMap;
	update = zoho.crm.updateRecord("Deals", dealId, updateMap);
	info update;
```

Json Format -- 


```json
{
  "Tag": [
    {
      "name": "TTUYTU",
      "id": "1136779000000802459"
    },
    {
      "name": "GDFGD",
      "id": "1136779000000802461"
    },
    {
      "name": "YUYJH",
      "id": "1136779000000802509"
    },
    {
      "name": "POP"
    },
    {
      "name": "POPPY"
    }
  ]
}
```

---

# Send Mail using Deluge :


```javascript
Method 1 (Without Template)- 

sendmail
[
	from :zoho.loginuserid
	to :recipientEmail
	subject :"Welcome to Our Service"
	message :"Hello " + "This is from test mail" + ",<br><br>Thank you for reaching out!"
	content type :HTML
]

Method (With Template)-

recipientEmail = "akashxwork@gmail.com";
templateId = "1136779000000857005";
templatedata = invokeurl
[
	url :"https://www.zohoapis.in/crm/v8/settings/email_templates/" + templateId
	type :GET
	connection:"getdata"
];

// info templatedata;
templatecontent = templatedata.get("email_templates").get("0").get("content");
// info templatecontent;
LeadName = "Akhri Pasta";
Email = recipientEmail;
revisedcontent = templatecontent.replaceAll("\$\{!Leads.Full_Name\}",LeadName).replaceAll("\$\{!Leads.Email\}",Email);
// info revisedcontent;
sendmail
[
from: zoho.adminuserid
to: recipientEmail
subject: "Welcome Email Template"
message: revisedcontent
]
```


---

# File Upload in Upload File ( Field ) :

```javascript
Note - Get file from attachments and add in file upload field in leads.

leadId = "1136779000000569198"; 

att = zoho.crm.getRelatedRecords("Attachments", "Leads", leadId);

file = invokeurl
[
	url: "https://www.zohoapis.in/crm/v2/Leads/" + leadId + "/Attachments/" + att.get(0).get("id")
	type: GET
	connection:"zohofile"
];
// info file;

file.setParamName("file");

resp = invokeurl 
[ 
	url: " https://www.zohoapis.in/crm/v2/files" 
	type: POST 
	files: file 
	connection: "zohofile" 
]; 
// info resp;

fileid = resp.get("data").get(0).get("details").get("id"); 
// info fileid;

fmp = Map(); 
fmp.put("file_id",fileid); 
fileList = List();
fileList.add(fmp); 
mp = Map(); 
mp.put("File_Upload",fileList); 
update = zoho.crm.updateRecord("Leads", leadId, mp);
info update;

```


---

# File Upload in Attachments :

```javascript
Note - Get file from upload file field and add in attachments in leads.

leadId = "1136779000000569198";
leadRecord = zoho.crm.getRecordById("Leads", leadId);
attId = leadRecord.get("File_Upload").getJSON("attachment_Id");
// info attId;

file = invokeurl
[
url :"https://www.zohoapis.in/crm/v2/Leads/" + leadId + "/Attachments/" + attId
type :GET
connection:"zohofile"
];

resp = zoho.crm.attachFile("Leads", leadId, file);
info resp;

```


---

# Take items from Deal Record (Sub-Form) and Create Element Record with Name & Category :

```javascript
console.log(".................START.................");

const stage = ZDK.Page.getField("Stage").getValue();

if (stage !== "Closed Won") {
    ZDK.Client.showAlert(`Deal must be "Closed Won" to proceed.`);
    // console.log ("It's not Closed Won");
    return;
}

const items = ZDK.Page.getSubform("Items").getValues();
// console.log(items);

let elementNameArray = [];
let elementCategoryArray = [];

if (items.length !== 0) {

items.forEach(item => {
    const prodName = item.Product_Name_1;
    const prodCategory = item.Product_Category;

    //     console.log(item);
    //     console.log(`prodName : ${prodName} ,  prodCategory : ${prodCategory}`);

    elementNameArray.push(prodName);
    elementCategoryArray.push(prodCategory);
});
} else {
    ZDK.Client.showAlert("There is no items in Sub-Form named items");
        // console.log("There is no items in Sub-Form named items");
        return;
}

if (elementNameArray.length > 0 && elementCategoryArray.length > 0) {
    const element = new ZDK.Apps.CRM.Elements.Models.Elements();

    element.Name = elementNameArray.join(", ");
    element.Element_Category = elementCategoryArray.join(", ");

    const res = ZDK.Apps.CRM.Elements.create(element);
    // console.log(res[0]._status);
    ZDK.Client.showMessage("Element record created successfully.");
} else {
    ZDK.Client.showAlert("Something went wrong!");
    return;
}

console.log(".................END.................");
```


---

# Set value (Amount Field) in Quotes Create Page :

```javascript
Note : Create quotes in deal record.

console.log(".................START.................");

const dealName = ZDK.Page.getField("Deal_Name").getValue();
// console.log(dealName.id);

var parentId = $Page.parent_record_id;
console.log(parentId);
console.log($Crm.user);
console.log($Page.record_id);

const dealData = ZDK.Apps.CRM.Deals.fetchById(dealName.id);
// console.log(dealData._Amount);

const amount = dealData._Amount;

if (amount !== null || amount !== undefined) {
    ZDK.Page.getField("Amount").setValue(amount);
} else {
    console.log("Something went wrong!")
}


console.log(".................END.................");
```


---

# Create Quotes and Show in Deals :

```javascript
Note - Take from Quotes Items & add products in same Deal Records(Both are Related List from Deal Record).

	dealId = "1136779000000569369";
	
	getProductData = zoho.crm.getRelatedRecords("Quotes", "Deals", dealId);
// 	info getProductData;
	products = getProductData.getJson("Product_Details");
// 	info products;
	
	
	productList = List();
	
	for each product in products
    {
		productId = product.get("product").get("id");
		mp = Map();
		update = zoho.crm.updateRelatedRecord("Products", productId, "Deals", dealId,mp);
		info update; 
    }
```


---

# Auto Age Calculate :

```javascript
// extract DOB
dobDay = 10;
dobMonth = 1;
dobYear = 2000;
now = zoho.currenttime;
// info now;
// extract current time
currentYear = now.getYear();
currentMonth = now.getMonth();
currentDay = now.getDay();
currentHour = now.getHour();
currentMinute = now.getMinutes();
currentSecond = now.getSeconds();
// calculate time
year = currentYear - dobYear;
month = currentMonth - dobMonth;
day = currentDay - dobDay;
if(month < 0)
{
	year = year - 1;
	month = month + 12;
}
if(day < 0)
{
	month = month - 1;
	if(month < 0)
	{
		year = year - 1;
		month = month + 12;
	}

	prevMonth = currentMonth - 1;
	if(prevMonth == 0)
	{
		prevMonth = 12;
	}

	daysInMonth = {1:31,2:28,3:31,4:30,5:31,6:30,7:31,8:31,9:30,10:31,11:30,12:31};
	day = day + daysInMonth.get(prevMonth);
}

age = year + "y " + month + "m " + day + "d " + " | " + currentHour + ":" + currentMinute + ":" + currentSecond;
info age;
```


---

# Create Invoice and Show in Deals(Transaction) :

```javascript
Note - Take Product which is shares(Related List) from Deal Record & create invoices(Same from Related List from Products - Shares).

// info zoho.crm.getRelatedRecords("Invoices", "Deals", "7171648000000613057");
// info zoho.crm.getRecordById("Invoices","7171648000000613109");
	
dealId = "7171648000000601064";
getDealData = zoho.crm.getRecordById("Deals",dealId);
shareName = getDealData.get("Share_Name");
// info getDealData;

if( shareName == null){
shareId = getDealData.get("Share_Name").get("id");
shareQuantity = getDealData.get("Share_Quantity");
sellRate = getDealData.get("Sell_Rate");
contactId = getDealData.get("Contact_Name").get("id");

getContactData = zoho.crm.getRecordById("Contacts",contactId);

contactCountry = getContactData.get("Mailing_Country");
contactState = getContactData.get("Mailing_State");
contactCity = getContactData.get("Mailing_City");
contactStreet = getContactData.get("Mailing_Street");
// info contactStreet;

shareList = List();
shareMap = Map();
shareMap.put("product",{"id":shareId});
shareMap.put("quantity",shareQuantity);
shareMap.put("unit_price",sellRate);
shareList.add(shareMap);
info shareList;

mp = Map();
mp.put("Subject", "Sample");
mp.put("Deal_Name__s", dealId);
mp.put("Product_Details", shareList);
mp.put("Billing_Street",contactStreet);
mp.put("Billing_City",contactCity);
mp.put("Billing_State",contactState);
mp.put("Billing_India",contactCountry);
mp.put("Shipping_Street",contactStreet);
mp.put("Shipping_City",contactCity);
mp.put("Shipping_State",contactState);
mp.put("Shipping_India",contactCountry);
info mp;

res = zoho.crm.createRecord("Invoices", mp);
info res;
}
else{
	info "nahi hoga";
}
```

Json Format -- Create Invoice and Show in Deals (Transaction) --

```json
{
  "Subject": "Sample",
  "Deal_Name__s": "7171648000000601064",
  "Product_Details": [
    {
      "product": {
        "id": "7171648000000610040"
      },
      "quantity": 90,
      "unit_price": 9876
    }
  ],
  "Billing_Street": "228 Runamuck Pl #2808",
  "Billing_City": "Baltimore",
  "Billing_State": "MD",
  "Billing_India": "United States",
  "Shipping_Street": "228 Runamuck Pl #2808",
  "Shipping_City": "Baltimore",
  "Shipping_State": "MD",
  "Shipping_India": "United States"
}
```


---

# Create a Duplicate List in Related List using Related Function in Leads (part-1):

```javascript
leadId = "1158761000000572119";
getLeadData = zoho.crm.getRecordById("Leads",leadId);
leadMobile = getLeadData.getJson("Mobile");
leadPhone = getLeadData.getJson("Phone");

// Check Lead Duplicate...
searchLead = zoho.crm.searchRecords("Leads","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
info searchLead;

// Check Contact Duplicate...
searchContact = zoho.crm.searchRecords("Contacts","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
info searchContact;

if(searchLead.size() > 1 || searchContact.size() > 0)
{
	relatedListXML = "<record>";
	n = 0;
	for each  leadRecord in searchLead
	{
		if(leadRecord.get("id") != leadId)
		{
			leadName = leadRecord.get("Full_Name");
			leadPhone = leadRecord.get("Mobile");
			relatedListXML = relatedListXML + "<row no='" + n + "'><FL val='Lead Name'>" + leadName + "</FL><FL val='Phone'>" + leadPhone + "</FL></row>";
			n = n + 1;
		}
	}
	for each  contactRecord in searchContact
	{
		contactName = contactRecord.get("Full_Name");
		contactPhone = contactRecord.get("Mobile");
		relatedListXML = relatedListXML + "<row no='" + n + "'><FL val='Contact Name'>" + contactName + "</FL><FL val='Phone'>" + contactPhone + "</FL></row>";
		n = n + 1;
	}
	relatedListXML = relatedListXML + "</record>";
}

return relatedListXML;
```

XML format --

```javascript
<record>
	<row no='0'>
		<FL val='Lead Name'>Sharma</FL>
		<FL val='Phone'>9876543210</FL>
	</row>
	<row no='1'>
		<FL val='Contact Name'>Sharma Ji</FL>
		<FL val='Phone'>9876543210</FL>
	</row>
	<row no='2'>
		<FL val='Contact Name'>Verma Ji</FL>
		<FL val='Phone'>9876543210</FL>
	</row>
</record>
```


---

# Remove all alphabet & special character using regex :

```javascript
	number = "$%^#$336356sjkdf456";
	info number.replaceAll("[^0-9]", "");


```


---

# Create a Duplicate List in Related List using Related Function in Leads (part-2) :

```javascript
string related_list.create_list_in_leads(String leadId)
{
// leadId = "1158761000000572119";
getLeadData = zoho.crm.getRecordById("Leads",leadId);
// info getLeadData;
leadMobile = getLeadData.getJson("Mobile");
leadPhone = getLeadData.getJson("Phone");
// Check Lead Duplicate...
searchLead = zoho.crm.searchRecords("Leads","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
// info searchLead;
// Check Contact Duplicate...
searchContact = zoho.crm.searchRecords("Contacts","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
info searchContact;
if(searchLead.size() > 1 || searchContact.size() > 0)
{
	relatedListXML = "<record>";
	// XML for Lead Category
	n = 0;
	for each  leadRecord in searchLead
	{
		if(leadRecord.get("id") != leadId)
		{
			lead_name = leadRecord.get("Full_Name");
			lead_phone = ifnull(leadRecord.get("Mobile"),leadRecord.get("Phone"));
			lead_id = leadRecord.get("id");
			lead_owner = leadRecord.get("Owner").get("name");
			row = "<row no='" + n + "'>";
			flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/" + lead_id + "'>" + lead_name + "</FL>";
			flRecord = "<FL val='Record'>Lead</FL>";
			flPhone = "<FL val='Phone'>" + lead_phone + "</FL>";
			flOwner = "<FL val='Owner'>" + lead_owner + "</FL>";
			relatedListXML = relatedListXML + row + flName + flPhone + flRecord + flOwner + "</row>";
			n = n + 1;
		}
	}
	// XML for Contact Category
	for each  contactRecord in searchContact
	{
		contact_name = contactRecord.get("Full_Name");
		contact_phone = ifnull(contactRecord.get("Mobile"),contactRecord.get("Phone"));
		contact_id = contactRecord.get("id");
		contact_owner = leadRecord.get("Owner").get("name");
		row = "<row no='" + n + "'>";
		flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/" + contact_id + "'>" + contact_name + "</FL>";
		flRecord = "<FL val='Record'>Contact</FL>";
		flPhone = "<FL val='Phone'>" + contact_phone + "</FL>";
		flOwner = "<FL val='Owner'>" + contact_owner + "</FL>";
		relatedListXML = relatedListXML + row + flName + flPhone + flRecord + flOwner + "</row>";
		n = n + 1;
	}
	relatedListXML = relatedListXML + "<row no='" + n + "'><FL val='Name'> Total - " + n + "</FL></row></record>";
	// info relatedListXML;
}
else
{
	relatedListXML = "<error><message>No Duplicate Contact</message></error>";
}
return relatedListXML;
}
```


XML format for Duplicate List in Related List --


```javascript
<record>
	<row no='0'>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/1158761000000593555'>Sharma</FL>
		<FL val='Record'>Lead</FL>
		<FL val='Phone'>9876543210</FL>
		<FL val='Owner'>Akash</FL>
	</row>
	<row no='1'>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/1158761000000617742'>Sharma Ji</FL>
		<FL val='Record'>Contact</FL>
		<FL val='Phone'>9876543210</FL>
		<FL val='Owner'>Akash</FL>
	</row>
	<row no='2'>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/1158761000000593575'>Verma Ji</FL>
		<FL val='Record'>Contact</FL>
		<FL val='Phone'>9876543210</FL>
		<FL val='Owner'>Akash</FL>
	</row>
	<row no='3'>
		<FL val='Name'> Total - 3</FL>
	</row>
</record>


```


---

# Create a Duplicate List in Related List using Related Function in Leads (part-3):

```javascript
leadData = zoho.crm.getRecordById("Leads",leadId);
if(leadData.containsKey("Mobile") && leadData.get("Mobile") != null)
{
	mobile = leadData.get("Mobile");
	search_lead = zoho.crm.searchRecords("Leads","(Mobile:equals:" + mobile + ")");
	// 	info search_lead;
	search_contact = zoho.crm.searchRecords("Contacts","(Mobile:equals:" + mobile + ")");
	// 	info contacts_data;
	if(search_lead.size() > 1 || search_contact.size() > 0)
	{
		relatedListXML = "<record>";
		count = 0;
		lead_count = 0;
		contact_count = 0;
		for each  leadRecord in search_lead
		{
			if(leadRecord.get("id") != leadId)
			{
				
				lead_name = leadRecord.get("Full_Name");
				lead_phone = ifnull(leadRecord.get("Mobile"),leadRecord.get("Phone"));
				lead_id = leadRecord.get("id");
				lead_owner = leadRecord.get("Owner").get("name");
				lead_status = leadRecord.get("Lead_Status");
				row = "<row no='" + count + "'>";
				flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/" + lead_id + "'>" + lead_name + "</FL>";
				flRecord = "<FL val='Record'>Lead</FL>";
				flPhone = "<FL val='Phone'>" + lead_phone + "</FL>";
				flOwner = "<FL val='Owner'>" + lead_owner + "</FL>";
				flStatus = "<FL val='Status'>" + lead_status + "</FL>";
				relatedListXML = relatedListXML + row + flRecord + flName + flPhone + flOwner + flStatus + "</row>";
				lead_count = lead_count + 1;
				count = count + 1;
			}
		}
		for each  contactRecord in search_contact
		{
			
			contact_name = contactRecord.get("Full_Name");
			contact_phone = ifnull(contactRecord.get("Mobile"),contactRecord.get("Phone"));
			contact_id = contactRecord.get("id");
			contact_owner = contactRecord.get("Owner").get("name");
			row = "<row no='" + count + "'>";
			flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/" + contact_id + "'>" + contact_name + "</FL>";
			flRecord = "<FL val='Record'>Contact</FL>";
			flPhone = "<FL val='Phone'>" + contact_phone + "</FL>";
			flOwner = "<FL val='Owner'>" + contact_owner + "</FL>";
			flStatus = "<FL val='Status'>—————</FL>";
			relatedListXML = relatedListXML + row + flRecord + flName + flPhone + flOwner + flStatus + "</row>";
			contact_count = contact_count + 1;
			count = count + 1;
		}
// 		relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Name'> Total - " + count + "</FL></row></record>";
		relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Name'> Lead - " + lead_count + " | " + "Contact - " + contact_count + "</FL></row></record>";
	}
	else
	{
		relatedListXML = "<error><message>No matching records found.</message></error>";
	}
	return relatedListXML;
}
else
{
	return "<error><message>No matching records found.</message></error>";
}
```


XML format --

```javascript
<record>
	<row no='0'>
		<FL val='Record'>Lead</FL>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/991021000006306009'>test 1 L</FL>
		<FL val='Phone'>919876543210</FL>
		<FL val='Owner'>Sales_IN</FL>
		<FL val='Status'>Fresh Lead</FL>
	</row>
	<row no='1'>
		<FL val='Record'>Lead</FL>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/991021000006306051'>test 2 L</FL>
		<FL val='Phone'>919876543210</FL>
		<FL val='Owner'>Sales_IN</FL>
		<FL val='Status'>Fresh Lead</FL>
	</row>
	<row no='2'>
		<FL val='Record'>Contact</FL>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/991021000006306093'>test 1 C</FL>
		<FL val='Phone'>919876543210</FL>
		<FL val='Owner'>Sales_IN</FL>
		<FL val='Status'>—————</FL>
	</row>
	<row no='3'>
		<FL val='Name'> Lead - 2 | Contact - 1</FL>
	</row>
</record>
```


---

# Get Files from Workdrive and Show in Related List using XML  (part-1) :


```javascript
string related_list.preview_workdrive_attachments(String leadId)
{
workdriveFolderId = "lpjr97c99675b349f4fa9bca794815e2defae";
header = Map();
header.put("Accept","application/vnd.api+json");
response = invokeurl
[
url :"https://www.zohoapis.in/workdrive/api/v1/files/" + workdriveFolderId + "/files"
type :GET
headers:header
connection:"workdrive"
];
files = response.get("data");
// info files.get("0").get("attributes") ;
if(files.size() > 0)
{
relatedListXML = "<record>";
count = 0;
for each file in files
{
name = file.get("attributes").get("name");
fileName = name.substring(0, name.lastIndexOf("."));
// info fileName;
fileId = file.get("id");
filePermalink = file.get("attributes").get("permalink");
fileDownloadURL = file.get("attributes").get("download_url");
filePreview = "https://workdrive.zoho.in/preview/" + fileId;
fileType = file.get("attributes").get("type");
fileSize = file.get("attributes").get("storage_info").get("size");

row = "<row no='" + count + "'>";
flName = "<FL val='File Name' link='true' url='" + filePermalink + "'>" + fileName + "</FL>";
flPreview = "<FL val='' link='true' url='" + filePreview + "'>Preview</FL>";
flDownload = "<FL val='' link='true' url='" + fileDownloadURL + "'>Download</FL>";
flType = "<FL val='Type'>" + fileType + "</FL>";
flSize = "<FL val='Size'>" + fileSize + "</FL>";

relatedListXML = relatedListXML + row + flName + flPreview + flType + flSize + "</row>";
count = count + 1;
}
relatedListXML = relatedListXML + "</record>";
// info relatedListXML;
}
else
{
relatedListXML = "<error><message>No WorkDrive Files/Folders</message></error>";
}
return relatedListXML;
}
```


XML format for Duplicate List in Related List --

```javascript
<record>
	<row no='0'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/lpjr9383ca3ec09f641d9912ea2bae983caf3'>Man Search for Meaning</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/lpjr9383ca3ec09f641d9912ea2bae983caf3'>Preview</FL>
		<FL val='Type'>pdf</FL>
		<FL val='Size'>316.21 KB</FL>
	</row>
	<row no='1'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/lpjr91e0ed4f7399340fc9acd949dab22e60d'>delugeRemainder</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/lpjr91e0ed4f7399340fc9acd949dab22e60d'>Preview</FL>
		<FL val='Type'>docs</FL>
		<FL val='Size'>25.20 KB</FL>
	</row>
	<row no='2'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/a1kl8dea62e91ac7349e8ac819c4cf4072c37'>AaeraCampaignleads</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/a1kl8dea62e91ac7349e8ac819c4cf4072c37'>Preview</FL>
		<FL val='Type'>spreadsheet</FL>
		<FL val='Size'>298.64 KB</FL>
	</row>
	<row no='3'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/a1kl897e4e1b5c5094368a121c1999a07d154'>Aaeraleads2025</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/a1kl897e4e1b5c5094368a121c1999a07d154'>Preview</FL>
		<FL val='Type'>spreadsheet</FL>
		<FL val='Size'>3.08 MB</FL>
	</row>
	<row no='4'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/a1kl857fc2863c2b04155b806eca6a76d44d1'>Vitamin_Cheat_Sheet</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/a1kl857fc2863c2b04155b806eca6a76d44d1'>Preview</FL>
		<FL val='Type'>pdf</FL>
		<FL val='Size'>53.88 MB</FL>
	</row>
	<row no='5'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/lpjr954f4b39f4076414aa87b1afcec819d4e'>The Monk Who Sold His Ferrari</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/lpjr954f4b39f4076414aa87b1afcec819d4e'>Preview</FL>
		<FL val='Type'>pdf</FL>
		<FL val='Size'>1.25 MB</FL>
	</row>
</record>
```


---

# Get Files from Workdrive and Show in Related List using XML  (part-2) :

```javascript
// caseId = "1043751000008644045";
caseData = zoho.crm.getRecordById("Cases",caseId);
// 	info caseData.get("Contact_Data_Link");
url = caseData.get("Contact_Data_Link");
if(url == null || url.isEmpty()){
return "<error><message>No WorkDrive Files/Folders</message></error>";
}
workdriveFolderId = url.subString(url.lastIndexOf("/") + 1);
// 	info workdriveFolderId;
header = Map();
header.put("Accept","application/vnd.api+json");
response = invokeurl
[
	url :"https://www.zohoapis.in/workdrive/api/v1/files/" + workdriveFolderId + "/files"
	type :GET
	headers:header
	connection:"contactworkdrive"
];

files = response.get("data");
// info files;
if(files.size() > 0)
{
	relatedListXML = "<record>";
	count = 0;
	for each  file in files
	{
		name = file.get("attributes").get("name");
		fileName = name.substring(0,name.lastIndexOf("."));
		// info fileName;
		fileId = file.get("id");
		filePermalink = file.get("attributes").get("permalink");
		fileDownloadURL = file.get("attributes").get("download_url");
		filePreview = "https://workdrive.zoho.in/preview/" + fileId;
		fileSize = file.get("attributes").get("storage_info").get("size");
		fileExtn = file.get("attributes").get("extn");
		fileType = "";
		if(fileExtn == "zip")
		{
			fileType = fileExtn;
		}
		else
		{
			fileType = file.get("attributes").get("type");
		}
		row = "<row no='" + count + "'>";
		flName = "<FL val='File Name' link='true' url='" + filePermalink + "'>" + fileName + "</FL>";
		flPreview = "<FL val='' link='true' url='" + filePreview + "'>Preview</FL>";
		flDownload = "<FL val='' link='true' url='" + fileDownloadURL + "'>Download</FL>";
		flType = "<FL val='Type'>" + fileType + "</FL>";
		flSize = "<FL val='Size'>" + fileSize + "</FL>";
// 		flUpload = "<FL val='Action' link='true' url='" + url + "'>► UPLOAD FILE</FL>";
		relatedListXML = relatedListXML + row + flName + flPreview + flType + flSize + "</row>";
		count = count + 1;
	}
		relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Type'>Action</FL><FL val='Size' link='true' url='" + url + "'>⬆️ Click to Upload</FL></row></record>";
	// info relatedListXML;
}
else
{
	relatedListXML = "<error><message>No WorkDrive Files/Folders</message></error>";
}

return relatedListXML;
```


---

# Get Files from Workdrive and Show in Related List using XML  (part-3) :
```javascript
// dealId = "1043751000009630359";
dealData = zoho.crm.getRecordById("Deals",dealId);
// 	info dealData.get("Contact_Data_Link");
url = dealData.get("Contact_Data_Link");
// info url;
if(url == null || url.isEmpty())
{
	return "<error><message>Please attach a WorkDrive link before proceeding.</message></error>";
}
workdriveFolderId = url.subString(url.lastIndexOf("/") + 1);
// info workdriveFolderId;

header = Map();
header.put("Accept","application/vnd.api+json");
response = invokeurl
[
	url :"https://www.zohoapis.in/workdrive/api/v1/files/" + workdriveFolderId + "/files"
	type :GET
	headers:header
	connection:"contactworkdrive"
];

files = response.get("data");
// info files;
if(files.size() > 0)
{
	relatedListXML = "<record>";
	count = 0;
	for each  file in files
	{
		name = file.get("attributes").get("name");
		fileName = name.substring(0,name.lastIndexOf("."));
		// info fileName;
		fileId = file.get("id");
		filePermalink = file.get("attributes").get("permalink");
		fileDownloadURL = file.get("attributes").get("download_url");
		filePreview = "https://workdrive.zoho.in/preview/" + fileId;
		fileSize = file.get("attributes").get("storage_info").get("size");
		fileExtn = file.get("attributes").get("extn");
		fileType = "";
		if(fileExtn == "zip")
		{
			fileType = fileExtn;
		}
		else
		{
			fileType = file.get("attributes").get("type");
		}
		row = "<row no='" + count + "'>";
		flName = "<FL val='File Name' link='true' url='" + filePermalink + "'>" + fileName + "</FL>";
		flPreview = "<FL val='' link='true' url='" + filePreview + "'>Preview</FL>";
		flDownload = "<FL val='' link='true' url='" + fileDownloadURL + "'>Download</FL>";
		flType = "<FL val='Type'>" + fileType + "</FL>";
		flSize = "<FL val='Size'>" + fileSize + "</FL>";
		// flUpload = "<FL val='File Name' link='true' url='" + url + "'>Upload</FL>";
		relatedListXML = relatedListXML + row + flName + flPreview + flType + flSize + "</row>";
		count = count + 1;
	}
	relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Type'>Action</FL><FL val='Size' link='true' url='" + url + "'>⬆️ Click to Upload</FL></row></record>";
	// info relatedListXML;
}
else
{
		relatedListXML = "<record><row no='0'><FL val=''>No file uploaded yet</FL><FL val='' link='true' url='" + url + "'>⬆️ Click here to upload a file</FL></row></record>";
}

return relatedListXML;
```


---

# Mobile Validator in Lead using Layout Rule :

```javascript
entityMap = crmAPIRequest.toMap().get("record");
mobile = entityMap.get("Mobile");
response = Map();
// if(ifNull(mobile, "").matches("^[0-9]{10}$") == false) 
if(mobile != "" && mobile.matches("^[0-9]{10}$") == false)
{
	response.put('status','error');
	response.put('message','Please enter a valid 10-digit mobile number.');
	return response;
}

return "";
```


---

# If Closed Won Stage is change to another Stage, it will gives error using Layout Rule :

```javascript
entityMap = crmAPIRequest.toMap().get("record");
stage = ifNull(entityMap.get("Stage"),"");
recordID = entityMap.get("id");
response = Map();
if(recordID == null)
{
	response.put('status','success');
}
else if(stage != "Closed Won")
{
	response.put('status','error');
	response.put('message','Not allowed to change stage');
}
else
{
	response.put('status','success');
}

return response;
```


---

# Auto Fill Pickup List in ClientScript :

```javascript
const service = ZDK.Page.getField("Practices_and_Services").getValue();
// console.log(service);

if (service) {
    const product = ZDK.Apps.CRM.Products.fetchById(service.id);
    console.log(product);
    const practice = product._Practice_Area;
    const subPractice = product._Sub_Practice_Area;
    console.log(practice);
    console.log(subPractice);
    // if (practice) {
        ZDK.Page.getField("Practice_Area").setValue(practice);
    // }
    // if (subPractice) {
        ZDK.Page.getField("Sub_Practice_Area").setValue(subPractice);
    // }
}
```


---

# Bulk Auto Update "Product Code" Field in Product Module :

```javascript
product_data = zoho.crm.getRecords("Products",0,200,{"cvid":785423000001339227});
for each  rec in product_data
{
	id = rec.get("id");
	// 	info id;
	Industry1 = rec.get("Industry1");
	Product_Category = rec.get("Product_Category");
	Product_Name = rec.get("Product_Name");
	Practice_Area = rec.get("Practice_Area");
	Sub_Practice_Area = rec.get("Sub_Practice_Area");
	Product_Code = Industry1 + "/" + Product_Category + "/" + Product_Name + "/" + Practice_Area + "/" + Sub_Practice_Area;
	Product_Code = Product_Code.replaceAll("/null","");
	Product_Code = Product_Code.replaceAll("null/","");
	res = zoho.crm.updateRecord("Products",id,{"Product_Code":Product_Code,"up":true});
}

```


---

# Find Duplicates in Leads & Contacts Module using Clientscript (Part - 1) :

```javascript
const leadMobile = ZDK.Page.getField("Mobile").getValue();
const leadOwner = ZDK.Page.getField("Owner").getValue();

const leads = ZDK.Apps.CRM.Leads.fetch();
const contacts = ZDK.Apps.CRM.Contacts.fetch();

////////////////////////////////////////////////////////////

let isLeadDuplicate = false;
let isContactDuplicate = false;

leads.forEach(lead => {
    const mobile = lead._Mobile;
    if (leadMobile && mobile && leadMobile === mobile) {
        isLeadDuplicate = true;
    }
});

contacts.forEach(contact => {
    const mobile = contact._Mobile;
    if (leadMobile && mobile && leadMobile === mobile) {
        isContactDuplicate = true;
    }
});

///////////////////////////////////////////////////////////

const isLeadDuplicate = leads.some(lead => {
    const mobile = lead._Mobile;
    return leadMobile && mobile && leadMobile === mobile;
});

const isContactDuplicate = contacts.some(contact => {
    const mobile = contact._Mobile;
    return leadMobile && mobile && leadMobile === mobile;
});

let duplicateName = "";

if (isLeadDuplicate && isContactDuplicate) {
    duplicateName = "Lead & Contact Record";
    ZDK.Client.showAlert(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isLeadDuplicate) {
    duplicateName = "Lead Record";
    ZDK.Client.showAlert(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isContactDuplicate) {
    duplicateName = "Contact Record";
    ZDK.Client.showAlert(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
}


```


---

# Find Duplicates in Leads & Contacts Module using Clientscript (Part - 2) :

```javascript
const leadMobile = ZDK.Page.getField("Mobile").getValue();
const leadMobileField = ZDK.Page.getField("Mobile");

const leadOwner = ZDK.Page.getField("Owner").getValue();

let isLeadDuplicate = false;
let isContactDuplicate = false;

try {
    const leads = await zrc.get('/crm/v8/Leads/search', {
        params: { phone: leadMobile, fields: 'Owner,Mobile' }
    });
    // console.log(leads.data.data);
    if (leads.data && leads.data.data.length > 0) {
        isLeadDuplicate = true;
    }
} catch (error) {
    console.log("Error fetching leads: " + error.message);
}

try {
    const contacts = await zrc.get('/crm/v8/Contacts/search', {
        params: { phone: leadMobile, fields: 'Owner,Mobile' }
    });
    if (contacts.data && contacts.data.data.length > 0) {
        isContactDuplicate = true;
    }
} catch (error) {
    console.log("Error fetching contacts: " + error.message);
}

let duplicateName = "";

if (isLeadDuplicate && isContactDuplicate) {
    duplicateName = "Lead & Contact Record";
    leadMobileField.showError(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isLeadDuplicate) {
    duplicateName = "Lead Record";
    leadMobileField.showError(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isContactDuplicate) {
    duplicateName = "Contact Record";
    leadMobileField.showError(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
}
```


---

# Update Status which have Blueprint transition using Deluge (Part - 1):

```javascript
caseId = "1043751000008644045";
	
response = invokeurl
[
	url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
	type: GET
	connection:"zoho_crm"
];

// info response.get("blueprint").get("transitions");

dataMap = Map();
dataMap.put("Notes", "Application Submitted");

blueprint1 = Map();
blueprint1.put("transition_id", "1043751000008644280"); // Application Submitted

decisionDate = zoho.currentdate.toString("yyyy-MM-dd");
dataMap.put("Decision_Due_by", decisionDate);

blueprint1.put("data", dataMap);
blueprintList = List();
blueprintList.add(blueprint1);
param = Map();
param.put("blueprint", blueprintList);
info param;
update = invokeurl
[
	url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
	type: PUT
	parameters: param.toString()
	connection:"zoho_crm"
];

info update;

```

Json Format --

```json
{
  "blueprint": [
    {
      "transition_id": "1043751000008644280",
      "data": {
        "Notes": "Application Submitted",
        "Decision_Due_by": "2026-01-13"
      }
    }
  ]
}
```


---

# Update Status to "Hold" which have Blueprint transition apart from "Visa Approved" or "Visa Rejected" using Deluge (Part - 2) :

```javascript
dealId = "1043751000007135808";
// 	caseId = "1043751000009263336";

// 	response = invokeurl
// [
// 	url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
// 	type: GET
// 	connection:"zoho_crm"
// ];
// info response;
	
get_related_data = zoho.crm.getRelatedRecords("Cases", "Deals", dealId);

if(get_related_data.size() > 0)
{
	for each record in get_related_data
	{
		caseId = record.get("id");
		currentStatus = record.get("Status");

		if(currentStatus != "Visa Approved" && currentStatus != "Visa Rejected" && currentStatus != "Hold")
		{
			dataMap = Map();
			// dataMap.put("Notes", "Hold");
			blueprint1 = Map();
			blueprint1.put("transition_id", "1043751000009328223"); // Hold
			decisionDate = zoho.currentdate.toString("yyyy-MM-dd");
			dataMap.put("Decision_Due_by", decisionDate);
			blueprint1.put("data", dataMap);
			blueprintList = List();
			blueprintList.add(blueprint1);
			param = Map();
			param.put("blueprint", blueprintList);
			info param;
				res = invokeurl
			[
				url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
				type: PUT
				parameters: param.toString()
				connection:"zoho_crm"
			];
			info res;
		}
	}
}
```


---

# Some Helpful Function :

```javascript
str = "one/two/three/four/four";
arr = str.toList("/");
arr.remove(arr.size() - 1);
arr = toString(arr);
arr = arr.replaceAll(",","/");
info arr;

number = "$%^#$33635328973326sjkdf456UJT";
info number.replaceAll("[^0-9]","");

num = "987654432101234";
if(num.length() > 10)
{
	num = num.subString(0,10);
}

info num;
```


---

# How to get Rollup Summary Field Value using Deluge :

```javascript
deal_id = "1158761000000660337";

response = invokeurl
[
url: "https://www.zohoapis.in/crm/v8/Deals/" + deal_id
type: GET
connection: "zohocrm"
];

info response.get("data").getJson("Rollup_Summary");

```


---

# Close all task in Case(application) inside an Deals Module :

```javascript
deal_id = "1043751000007110324";
deal_related = zoho.crm.getRelatedRecords("Cases","Deals",deal_id);
for each  case in deal_related
{
	status = case.getJSon("Status");
	app_related = zoho.crm.getRelatedRecords("Tasks","Cases",case.getJson("id"));
	if(status == "Hold" && app_related.size() > 0)
	{
		for each  rec in app_related
		{
			if(rec.get("Status") != "Completed")
			{
				update = zoho.crm.updateRecord("Tasks",rec.getJson("id"),{"Status":"Completed"});
				// info "task : " + update;
			}
		}
	}
}
```

---

# Create Multiple Deal with certain condition (Multiple Countries) :

```javascript
// Fetch deal record
dealRecord = zoho.crm.getRecordById("Deals", id);
countriesOfInterest = dealRecord.get("Country_of_Interest");
isFirstCountry = true;

// Check if multiple countries exist
if(countriesOfInterest.size() > 1)
{
    // Process each country
    for each country in countriesOfInterest
    {
        // Generate deal name components
        intakeDate = dealRecord.get("Possible_Intake");
        formattedMonthYear = intakeDate.toDate().toString("MMM yyyy");
        originalDealName = dealRecord.get("Deal_Name");
        updatedDealName = originalDealName + "/" + country + "/" + formattedMonthYear;
        
        // Create single-item country list
        singleCountryList = List();
        singleCountryList.add(country);
        
        if(isFirstCountry)
        {
            // Update existing deal record
            updateData = Map();
            updateData.put("Country_of_Interest", singleCountryList);
            updateData.put("Deal_Name", updatedDealName);
            updateResponse = zoho.crm.updateRecord("Deals", id, updateData);
            
            isFirstCountry = false;
        }
        else
        {
            // Clone existing deal record
            clonedDealData = dealRecord.toMap();
            
            // Remove system fields that shouldn't be copied
            clonedDealData.remove("id");
            clonedDealData.remove("Owner");
            clonedDealData.remove("Created_Time");
            clonedDealData.remove("Modified_Time");
            clonedDealData.remove("Created_By");
            clonedDealData.remove("Modified_By");
            clonedDealData.remove("Tag");
            
            // Create new deal record
            createResponse = zoho.crm.createRecord("Deals", clonedDealData);
            
            // Update the newly created deal
            if(createResponse.get("id") != null)
            {
                newDealId = createResponse.get("id");
                
                updateData = Map();
                updateData.put("Country_of_Interest", singleCountryList);
                updateData.put("Deal_Name", updatedDealName);
                updateResponse = zoho.crm.updateRecord("Deals", newDealId, updateData);
            }
        }
    }
}
else
{
    // Handle single country scenario
    currentDealName = dealRecord.get("Deal_Name");
    country = dealRecord.get("Country_of_Interest");
    intakeDate = dealRecord.get("Possible_Intake");
    formattedMonthYear = intakeDate.toDate().toString("MMM yyyy");
    updatedDealName = currentDealName + "/" + country + "/" + formattedMonthYear;
    
    // Update deal name only
    updateData = Map();
    updateData.put("Deal_Name", updatedDealName);
    updateResponse = zoho.crm.updateRecord("Deals", id, updateData);
}
```


---

# Update URL from Contact to Deal to Application(Case) Module if there is not link in contact then it will create :

```javascript
parentFolderId = "mp021cbba2181fc654b57b77c7a24cd2e87dd";
// dealId = "1043751000007135808";
caseData = zoho.crm.getRelatedRecords("Cases","Deals",dealId);
dealData = zoho.crm.getRecordById("Deals",dealId);
dealURL = dealData.get("Contact_Data_Link");

isUpdated = false;

if(dealURL != null && !dealURL.isEmpty())
{
	if(caseData != null && caseData.size() > 0)
	{
		// isUpdated = false;

		for each caseRec in caseData
		{
			caseId = caseRec.get("id");
			caseLink = caseRec.get("Contact_Data_Link");

			if(caseLink == null || caseLink.isEmpty())
			{
				updateMap = Map();
				updateMap.put("Contact_Data_Link",dealURL);
				zoho.crm.updateRecord("Cases",caseId,updateMap);
				isUpdated = true;
			}
		}

		if(isUpdated)
		{
			return "Url attached to Application Record (Related List)";
		}
		else
		{
			return "Application already has Contact Data Link";
		}
	}
	else
	{
		return "No Application found for this Deal";
	}
}	

contactNameFromDeal = dealData.get("Contact_Name");
if(contactNameFromDeal == null)
{
	return "No Contact linked to Deal";
}

contactId = contactNameFromDeal.get("id");
contactData = zoho.crm.getRecordById("Contacts",contactId);
contactName = contactData.get("Full_Name");
contactURL = contactData.get("Contact_Data_Link1");

if(contactURL != null && !contactURL.isEmpty())
{
// update deal "Contact_Data_Link"....
	updateDealMap = Map();
	updateDealMap.put("Contact_Data_Link",contactURL);
	zoho.crm.updateRecord("Deals",dealId,updateDealMap);
// update contact "Contact_Data_Link1"....
	if(caseData != null && caseData.size() > 0)
	{
		for each caseRec in caseData
		{
			caseId = caseRec.get("id");
			caseLink = caseRec.get("Contact_Data_Link");

			if(caseLink == null || caseLink.isEmpty())
			{
				updateMap = Map();
				updateMap.put("Contact_Data_Link",contactURL);
				zoho.crm.updateRecord("Cases",caseId,updateMap);
			}
		}
		return "Contact module link synced to Deal & Application :- " + contactURL;
	}
	else
	{
		return "Contact module link synced to Deal :- " + contactURL;
	}
}
else
{
	// create folder in workdrive...
	folderName = contactName + "_" + contactId;
	createFolderResp = zoho.workdrive.createFolder(folderName,parentFolderId,"contactworkdrive");
	if(!createFolderResp.containKey("data"))
	{
		return "Something went wrong";
	}
	folderPermalink = createFolderResp.get("data").get("attributes").get("permalink");
	// 	info folderPermalink;
	// update contact "Contact_Data_Link1"....
	updateContactMap = Map();
	updateContactMap.put("Contact_Data_Link1",folderPermalink);
	zoho.crm.updateRecord("Contacts",contactId,updateContactMap);
	// update deal "Contact_Data_Link....
	updateDealMap = Map();
	updateDealMap.put("Contact_Data_Link",folderPermalink);
	zoho.crm.updateRecord("Deals",dealId,updateDealMap);
	// update application "Contact_Data_Link....
	if(caseData != null && caseData.size() > 0)
	{
		for each  caseRec in caseData
		{
			caseId = caseRec.get("id");
			caseLink = caseRec.get("Contact_Data_Link");
			if(caseLink == null || caseLink.isEmpty())
			{
				updateMap = Map();
				updateMap.put("Contact_Data_Link",folderPermalink);
				zoho.crm.updateRecord("Cases",caseId,updateMap);
			}
		}
	}
	return "Url attached to Contact, Deal & Application";
}
```


---

# Create a new call record using deluge (Part - 1):

```javascript
number = "+917987322610";
dateString = "13/01/2026";
timeString = "17:16:50";
dateValue = dateString.toDate("dd/MM/yyyy");
dateTimeString = dateString + " " + timeString;
dateTimeValue = dateTimeString.toTime("dd/MM/yyyy HH:mm:ss");
isoDateTime = dateTimeValue.toString("yyyy-MM-dd'T'HH:mm:ssXXX");
mp = Map();
mp.put("Subject","Missed call from " + number);
mp.put("Call_Type","Missed");
// mp.put("Call_Duration","0:31");
mp.put("$se_module","Leads");
mp.put("Call_Status","Missed");
mp.put("Dialled_Number",number);
mp.put("Call_Start_Time",isoDateTime);
// mp.put("","");
create = zoho.crm.createRecord("Calls", mp);
info create;

```


---

# Create a new call record using deluge (Part - 2):

```javascript
mobile = "+911234567890";
isoDateTime = now.toString("yyyy-MM-dd'T'HH:mm:ssXXX");

dataMap = Map();
dataMap.put("Subject", "Missed call from " + mobile);
dataMap.put("Call_Type", "Missed");
dataMap.put("Call_Status", "Missed");
dataMap.put("Dialled_Number", mobile);
dataMap.put("Call_Start_Time", isoDateTime);
dataMap.put("To_Number__s", mobile);

dataList = List();
dataList.add(dataMap);

payload = Map();
payload.put("data", dataList);

response = invokeurl
[
    url :"https://www.zohoapis.in/crm/v8/Calls"
    type : POST
    parameters : payload.toString()
	connection : "zohocrm"
];

info response;

```


---

# Get data from CRM and Show in redirect HTML Page in ZOHO CREATOR :

OnLoad -

```javascript
// Display Company Logo
image_url = "https://www.abhinav.com/wp-content/uploads/2024/08/Ahinav-new-logo-png.png";
input.plain = "<div style=\"text-align:center;\"><img src=\"" + image_url + "\" style=\"height:50px;\" alt=\"Logo\"></div>";

// Hide DOB Field & Submit Button
hide Date_of_Birth;
input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";

```

workflow function -


```javascript
mobile = input.Registered_Mobile;
dob = input.Date_of_Birth;
contact_id = list();
if(mobile.startsWith("+91"))
{
	mobile = mobile.substring(3);
}

// First search: Mobile
critera = "(Mobile:equals:" + mobile + ")";
search_contact = zoho.crm.searchRecords("Contacts",critera);
if(search_contact.size() > 0)
{
	contact_dob = search_contact.get(0).get("Date_of_Birth");
	if(contact_dob != null && contact_dob == dob.toString("yyyy-MM-dd"))
	{
		contact_id = search_contact.get(0).get("id").toLong();
	}
}

page_url = "https://creatorapp.zohopublic.com/sanjay645/application-tracker/page-perma/Show_Deal_And_Process_stage/wa8J2R1JXevb4J7naY7BRpDUkO7wnFPjxsJmwfBxT6Gj0ZUmnwDz9uSuHPjm2xDKZJKCDanx5VT7Og7ss5shyYYTT7WOR7df1nOp" + "?uid=" + contact_id;
openUrl(page_url,"same window");

// + if(search_with_dob.size() > 0,"","&bd=" + isDob);
//  + "&bd=" + isDob;

// mobile = zoho.encryption.base64Encode(mobile);
// openUrl("#Page:Show_Deal_And_Process_stage?Registered_Mobile=" + mobile,"same window");

// detail = https://creatorapp.zohopublic.com/sanjay645/application-tracker/form-perma/Customer_Detail/DxkPY6WrhORFuajTa7UgNSFPwdgukrqAbDSsWHT3Eq5PfXstBZPenmKQXuyZQS6FEVyxGPxaGMy09WEnAS7fvdB36EKms3X0FOnJ

//   page = https://creatorapp.zohopublic.com/sanjay645/application-tracker/page-perma/Show_Deal_And_Process_stage/wa8J2R1JXevb4J7naY7BRpDUkO7wnFPjxsJmwfBxT6Gj0ZUmnwDz9uSuHPjm2xDKZJKCDanx5VT7Og7ss5shyYYTT7WOR7df1nOp

```

Checking If Moile is null or not ( function ) -


```javascript
if(input.Checkbox.isEmpty())
{
	hide Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";
	enable Registered_Mobile;
	return;
}

mobile = input.Registered_Mobile;
if(mobile == null || mobile.trim() == "")
{
	alert "Mobile number is required to proceed. Please enter it first.";
	input.Checkbox = List();
	enable Registered_Mobile;
	return;
}

if(mobile.startsWith("+91"))
{
	mobile = mobile.substring(3);
}

if(ifNull(mobile,"").matches("^[0-9]{10}$") == false)
{
	alert "Please enter a valid 10-digit mobile number.";
	input.Checkbox = List();
	enable Registered_Mobile;
	return;
}

disable Registered_Mobile;
criteria = "(Mobile:equals:" + mobile + ")";
search_contact = zoho.crm.searchRecords("Contacts",criteria);
if(search_contact.size() == 0)
{
	alert "No account is associated with the provided mobile number.";
	hide Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";
	input.Checkbox = List();
	input.Registered_Mobile = "";
	enable Registered_Mobile;
	return;
}

contact = search_contact.get(0);
contact_dob = contact.get("Date_of_Birth");
contact_owner = contact.get("Owner").get("name");
if(contact_dob == null)
{
	alert "Date of Birth is missing. Please contact your account manager - " + contact_owner;
	hide Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";
	input.Checkbox = List();
	input.Registered_Mobile = "";
	enable Registered_Mobile;
}
else
{
	show Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{}</style>";
}
```

html snippet ( redirect ) -

```javascript
<%{
	// 	uid = zoho.encryption.base64Decode(uid); 
	// 	criteria = "(Mobile:equals:" + uid + ")";
	// 	search_contact = zoho.crm.searchRecords("Contacts",criteria);
	// 	related_deals = if(search_contact.size() > 0,zoho.crm.getRelatedRecords("Deals","Contacts",search_contact.get(0).get("id").toLong()),list());	
	// 	form_url = "#Form:Customer_Detail";

	input.uid = input.uid;
	related_deals = if(input.uid.isNumber(),zoho.crm.getRelatedRecords("Deals","Contacts",input.uid.toLong()),list());

	form_url = "https://creatorapp.zohopublic.com/sanjay645/application-tracker/form-perma/Customer_Detail/DxkPY6WrhORFuajTa7UgNSFPwdgukrqAbDSsWHT3Eq5PfXstBZPenmKQXuyZQS6FEVyxGPxaGMy09WEnAS7fvdB36EKms3X0FOnJ";

	%>

<div style="font-family: 'Segoe UI', Arial, sans-serif; background-color: #f4f7f6; padding: 25px; border-radius: 12px;">
    <h1 style="color: #333; text-align: center; margin-bottom: 25px;">Application Details</h1>
	<div style="margin-bottom: 20px;">
	<!-- BACK TO SEARCH BUTTON -->
    <a href=<%=form_url%> style="text-decoration: none;">
        <button style="padding: 10px 20px; background-color: #6c757d; color: white; border: none; border-radius: 5px; cursor: pointer;">
            ← Back to Search
        </button>
    </a>
</div>

<%

	if(related_deals.size() > 0)
	{
		for each  deal in related_deals
		{
			deal_id = deal.get("id").toLong();
			// related_cases = zoho.crm.getRelatedRecords("Cases","Deals",deal_id);
			related_cases_raw = zoho.crm.getRelatedRecords("Cases","Deals",deal_id);
			sortKeys = List();
			caseMap = Map();
			related_cases = List();
			for each  rec in related_cases_raw
			{
				sortVal = ifnull(rec.get("Subject"),"") + "_" + rec.get("id");
				sortKeys.add(sortVal);
				caseMap.put(sortVal,rec);
			}
			sortKeys.sort(true);
			for each  key in sortKeys
			{
				related_cases.add(caseMap.get(key));
			}

			%>

<div style="background-color: #ffffff; padding: 20px; border-radius: 10px; border-left: 6px solid #20B2E3; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 30px;">
        <h2 style="margin-top: 0; color: #20B2E3; font-size: 20px; border-bottom: 1px solid #eee; padding-bottom: 10px;">Deal: <%=deal.get("Deal_Name")%></h2>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 12px; font-size: 15px; padding-top: 10px;">
            <div><strong>Deal Stage:</strong> <%=deal.get("Stage")%></div>
            <div><strong>Deal Owner:</strong> <%=ifnull(deal.get("Owner").get("name"),"N/A")%></div>
        </div>

        <div style="display: flex; gap: 20px; margin-top: 25px; flex-wrap: wrap;">
            
            <!-- OPEN COLUMN -->
            <div style="flex: 1; min-width: 300px;">
                <h3 style="color: #28a745; font-size: 16px; border-bottom: 2px solid #28a745; padding-bottom: 5px;">OPEN PROCESSES</h3>
<%
			open_count = 0;
			for each  case_rec in related_cases
			{
				if(case_rec.get("Status") != "Closed")
				{
					open_count = open_count + 1;
					%>
<div style="background-color: #e8f5e9; border: 1px solid #c8e6c9; padding: 12px; border-radius: 6px; margin-top: 10px;">
                    <div style="font-weight: bold; color: #2e7d32;"><%=case_rec.get("Subject")%></div>
                    <div style="font-size: 13px; margin-top: 4px;"><strong>Status:</strong> <%=case_rec.get("Status")%></div>
                    <div style="font-size: 13px;"><strong>Owner:</strong> <%=ifnull(case_rec.get("Owner").get("name"),"N/A")%></div>
                </div>
<%
				}
			}
			if(open_count == 0)
			{
				%>
<div style="color:#999; padding-top:10px; font-style:italic;">No open processes.</div>

<%
			}
			%>
</div>

            <!-- CLOSED COLUMN -->
            <div style="flex: 1; min-width: 300px;">
                <h3 style="color: #dc3545; font-size: 16px; border-bottom: 2px solid #dc3545; padding-bottom: 5px;">CLOSED PROCESSES</h3>
<%
			closed_count = 0;
			for each  case_rec in related_cases
			{
				if(case_rec.get("Status") == "Closed")
				{
					closed_count = closed_count + 1;
					%>
<div style="background-color: #FFF5F5; border: 1px solid #f5c6cb; padding: 12px; border-radius: 6px; margin-top: 10px;">
                    <div style="font-weight: bold; color: #721c24;"><%=case_rec.get("Subject")%></div>
                    <div style="font-size: 13px; margin-top: 4px;"><strong>Status:</strong> <%=case_rec.get("Status")%></div>
                    <div style="font-size: 13px;"><strong>Owner:</strong> <%=ifnull(case_rec.get("Owner").get("name"),"N/A")%></div>
                </div>
<%
				}
			}
			if(closed_count == 0)
			{
				%>
<div style="color:#999; padding-top:10px; font-style:italic;">No closed processes.</div>

<%
			}

			%>
</div>

        </div>
    </div>

<%
		}
	}
	else
	{
		%>
<div style="text-align: center; padding: 40px; background-color: #ffe5e5; border-radius: 12px; border: 1px solid #ff6b6b; color: #c62828; font-family: 'Segoe UI', Arial, sans-serif; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <h2 style="margin-top: 0; color: #c62828;">Details Not Found</h2>
    <p style="font-size: 16px; margin: 15px 0;">
        The mobile number or the date of birth entered seems to be incorrect.
    </p>
    <p style="font-size: 14px; color: #8e0000;">
        Please double-check your information and try again.
    </p>
</div>

<%
	}
	%>
</div>
<%

}%>
```


---

# Auto Lead Conversion in Contacts, Acocunts and Deals Module :

```javascript
// leadId = "6103868000006230215";
lead = zoho.crm.getRecordById("Leads",leadId);
if(lead.get("Lead_Status") == "Qualified")
{
	// For Contact
	contactMap = Map();
	contactMap.put("First_Name",lead.get("First_Name"));
	contactMap.put("Last_Name",lead.get("Last_Name"));
	contactMap.put("Email",lead.get("Email"));
	contactMap.put("Mobile",lead.get("Mobile"));
	contactMap.put("Mailing_Street",lead.get("Street"));
	contactMap.put("Mailing_City",lead.get("City"));
	contactMap.put("Mailing_State",lead.get("State"));
	contactMap.put("Mailing_Zip",lead.get("Zip_Code"));
	contactMap.put("Mailing_Country",lead.get("Country"));

	// 	For Account
	accountMap = Map();
	accountMap.put("Account_Name",ifnull(lead.get("Company"),lead.get("Full_Name")));
	accountMap.put("Email",lead.get("Email"));
	accountMap.put("Mobile",lead.get("Mobile"));
	accountMap.put("Billing_Street",lead.get("Street"));
	accountMap.put("Billing_City",lead.get("City"));
	accountMap.put("Billing_State",lead.get("State"));
	accountMap.put("Billing_Zip",lead.get("Zip_Code"));
	accountMap.put("Billing_Country",lead.get("Country"));

	// For Deal
	dealMap = Map();
	dealMap.put("Deal_Name",lead.get("Last_Name"));
	dealMap.put("Stage","Qualification");

	// Conversion
	convertMap = Map();
	convertMap.put("Accounts",accountMap);
	convertMap.put("Contacts",contactMap);
	convertMap.put("Deals",dealMap);
	
	convert = zoho.crm.convertLead(leadId,convertMap);
	info convert;
}

```


---

# ZET Code for CRM (Async Vesion) Part - 1 :

```javascript

console.log("......First......");

ZOHO.embeddedApp.on("PageLoad", async function (data) {
    try {
        console.log("PageLoad data:", data);

        const taskId = data.EntityId;
        const module = data.Entity;

        if (!taskId || !module) {
            throw new Error("Missing CRM context");
        }

        // await ONLY the API call
        const response = await ZOHO.CRM.API.getRecord({
            Entity: module,
            RecordID: taskId
        });

        console.log("CRM Response:", response);

        const description = response.data[0].Description;
        // renderChat(description);

    } catch (error) {
        console.error("Widget error:", error);
        document.getElementById("title").innerText =
            "Failed to load conversation";
    }
});

ZOHO.embeddedApp.init();

console.log("......Last......");

```


---

# ZET Code for CRM (.then Vesion) Part - 2 :

```javascript

console.log("......First......");

ZOHO.embeddedApp.on("PageLoad", function (data) {

    const taskId = data.EntityId;
    const module = data.Entity;

    ZOHO.CRM.API.getRecord({
        Entity: module,
        RecordID: taskId
    })
    .then(function (response) {
        console.log(response);
    })
    .catch(function (error) {
        console.error(error);
    });

});

ZOHO.embeddedApp.init();

console.log("......Last......");

```

---