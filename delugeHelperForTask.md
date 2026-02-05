# ZOHO CRM TASK AUTOMATION TEMPLATES

## Template 1: Basic Task with LAR_ID
**Use when:** Need workflow trigger, no owner assignment
```javascript
currentDateTime = zoho.currenttime;
reminderTime = currentDateTime.addMinutes(15);
taskMap = Map();
taskMap.put("Who_Id",idd);
taskMap.put("What_Id",id);
taskMap.put("$se_module","Cases");
taskMap.put("Subject","[YOUR SUBJECT]");
taskMap.put("Status","Not Started");
taskMap.put("Priority","High");
taskMap.put("Due_Date",zoho.currentdate);
taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + reminderTime.toString("yyyy-MM-dd'T'HH:mm:ssXXX")});
createdTask = zoho.crm.createRecord("Tasks",taskMap,{"lar_id":"[YOUR_LAR_ID]","trigger":{"workflow","blueprint","approval"}});
info createdTask;
```

---

## Template 2: Task with Owner
**Use when:** Need to assign to case owner, no workflow trigger
```javascript
app_data = zoho.crm.getRecordById("Cases",id);
owner_id = app_data.get("Owner").get("id");
currentDateTime = zoho.currenttime;
reminderTime = currentDateTime.addMinutes(15);
taskMap = Map();
taskMap.put("Who_Id",idd);
taskMap.put("What_Id",id);
taskMap.put("$se_module","Cases");
taskMap.put("Owner",owner_id);
taskMap.put("Subject","[YOUR SUBJECT]");
taskMap.put("Status","Not Started");
taskMap.put("Priority","High");
taskMap.put("Due_Date",zoho.currentdate);
taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + reminderTime.toString("yyyy-MM-dd'T'HH:mm:ssXXX")});
createdTask = zoho.crm.createRecord("Tasks",taskMap);
info createdTask;
```

---

## Template 3: Multiple Tasks (Owner + LAR_ID)
**Use when:** Need to create related tasks - one for owner, one with workflow
```javascript
app_data = zoho.crm.getRecordById("Cases",id);
owner_id = app_data.get("Owner").get("id");
currentDateTime = zoho.currenttime;
reminderTime = currentDateTime.addMinutes(15);
taskMap = Map();
taskMap.put("Who_Id",idd);
taskMap.put("What_Id",id);
taskMap.put("$se_module","Cases");
taskMap.put("Status","Not Started");
taskMap.put("Priority","High");
taskMap.put("Due_Date",zoho.currentdate);
taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + reminderTime.toString("yyyy-MM-dd'T'HH:mm:ssXXX")});

// Task 1: With Owner
taskMap.put("Owner",owner_id);
taskMap.put("Subject","[YOUR SUBJECT 1]");
createdTask1 = zoho.crm.createRecord("Tasks",taskMap);
info createdTask1;

// Task 2: With LAR_ID
taskMap.remove("Owner");
taskMap.put("Subject","[YOUR SUBJECT 2]");
createdTask2 = zoho.crm.createRecord("Tasks",taskMap,{"lar_id":"[YOUR_LAR_ID]","trigger":{"workflow","blueprint","approval"}});
info createdTask2;
```

---

## CUSTOMIZATION OPTIONS

### Due Date Variations
```javascript
// Same Day (default)
taskMap.put("Due_Date",zoho.currentdate);

// Next Day
dueDate = zoho.currentdate.addDay(1);
taskMap.put("Due_Date",dueDate);

// Custom Days
dueDate = zoho.currentdate.addDay(3);
taskMap.put("Due_Date",dueDate);
```

### Reminder Time Variations
```javascript
// 15 Minutes from now (default)
currentDateTime = zoho.currenttime;
reminderTime = currentDateTime.addMinutes(15);
taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + reminderTime.toString("yyyy-MM-dd'T'HH:mm:ssXXX")});

// Fixed Time (10:00 AM)
dueDate = zoho.currentdate.addDay(1);
reminderTime = dueDate.toString("yyyy-MM-dd") + "T10:00:00+05:30";
taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + reminderTime});

// Custom Minutes
reminderTime = currentDateTime.addMinutes(30);
taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + reminderTime.toString("yyyy-MM-dd'T'HH:mm:ssXXX")});
```

---

## COMMON LAR_IDs
```
1043751000001576183  →  Refund Application
1043751000001576434  →  Visa Follow-up
1043751000001576328  →  Interview Required
```

---

## QUICK DECISION GUIDE

| Need | Template |
|------|----------|
| Workflow trigger only | Template 1 |
| Assign to case owner | Template 2 |
| Two related tasks | Template 3 |
| Fixed time reminder | Use customization option |
| Next day due date | Use customization option |

---

**Simply copy, paste, and replace `[YOUR SUBJECT]` and `[YOUR_LAR_ID]` placeholders!**