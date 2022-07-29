# Download-and-Upload-Files-From-and-To-ZohoCRM-File-Upload-Fields
A Deluge code for downloading and uploading files from and to Zoho CRM file upload fields.

## Problem Statement
This is something that we find ourselves occasionally doing and the lack of clear documentations online warrants a quick article. In the most recent use case, we needed to move record info from one module to another as part of a migration task. This involves moving files from file upload fields across modules.

## Scopes Needed
- ZohoCRM.modules.ALL
- ZohoCRM.Files.CREATE
- ZohoCRM.modules.attachments.all

## Tutorial
- Get the file upload field
- If not null, iterate over every file in the field and
  - Download the file via the attachment id
  - Set param name to file
  - Upload the file to Zoho File Systems (ZFS)
  - Get the file ID and put it in a map, in a list
  - Upload to another file upload field by putting the list into the update map

```
recordId_1 = xxxxxxxxxxxxxxxxx;
recordInfo = zoho.crm.getRecordById("Module_1",recId);
// Get file upload field
fileUploadField = recordInfo.get("File_Upload_Field_1");
if (!fileUploadField.isNull())
{
	// Init
	fileIdList = List();
	for each f in fileUploadField
	{
		// Download the file via the attachment id
		file = invokeurl
		[
			url: "https://www.zohoapis.com/crm/v2/ServiceCenters/"+recId+"/Attachments/"+f.get("attachment_Id")
			type: GET
			connection:"crm"
		];
		//  Set param name to file
		file.setparamname("file");
		// Upload to ZFS
		upload = invokeurl 
		[ 
			url: " https://www.zohoapis.com/crm/v2/files" 
			type: POST 
			files: file 
			connection: "crm" 
		]; 
		// Get the file Id
		fileId = upload.get("data").get(0).get("details").get("id"); 
		// Add to fileIdList
		fileIdList.add({"file_id":fileId});
	}
	info fileIdList;
	// Upload to another file upload field
	mp = Map(); 
	mp.put("File_Upload_Field_2",fileIdList); 
	update = zoho.crm.updateRecord("Module_2", xxxxxxxxxxxxxxxxx, mp);
	info update;
}
```

## Additional Tip - How to Delete Files from File Upload Field
While we're on the topic of file upload fields, here's a quick code you can use to delete files from a file upload field. A recent use case for this is an automation where we needed to replace a contract document with the signed copy when the contract gets signed.

```
filelist = List();
// You can add new files here
// <script>
// Here is where you delete existing files
for each existingFile in fileUploadField
{
	existingFileId = existingFile.get("attachment_Id");
	mp = Map();
	mp.put("attachment_id",existingFileId);
	mp.put("_delete",null);
	filelist.add(mp);
}
// Update the file upload field
mp = Map(); 
mp.put("File_Upload_Field",fileIdList); 
update = zoho.crm.updateRecord("Module_2", xxxxxxxxxxxxxxxxx, mp);
info update;
```

