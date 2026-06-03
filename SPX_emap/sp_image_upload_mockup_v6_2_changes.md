# Changes: sp_image_upload_mockup_v6_2.html

## File: sp_image_upload_mockup_v6_2.html — Line ~716

### Change
Shortened the warning message in the "Unsupported file formats" section.

### Before
```js
+'<div class="alert-warn" style="margin-bottom:10px"><div class="aw-body">Only JPG, PNG and HEIC files can be uploaded. The '
+badFiles.length+' file'+(badFiles.length>1?'s below are':' below is')+' not supported and will be skipped automatically — everything else will upload as normal.</div></div>'
```

### After
```js
+'<div class="alert-warn" style="margin-bottom:10px"><div class="aw-body">Only JPG, PNG and HEIC files can be uploaded.</div></div>'
```
