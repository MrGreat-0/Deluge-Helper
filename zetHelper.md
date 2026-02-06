### Related Repository
[📄 Transcript Viewer](https://github.com/MrGreat-0/Transcript_Viewer)  
A custom Zoho CRM widget for parsing and visualizing JSON-based call transcripts.


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