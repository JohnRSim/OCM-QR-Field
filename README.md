![QR-Generator](https://user-images.githubusercontent.com/364208/190979725-0c1e4641-80bb-4503-b4e5-6d5070bae4ea.png)


# OCM-QR-Field (In Development)

Free Custom Component built with Svelte for generating QR Codes; installed as a component and used directly within an OCM Form.



https://user-images.githubusercontent.com/364208/191071289-0b733cf5-fcfc-46fd-8c4c-feae37817c34.mp4



# Data Type "Large Text"
This field stores json data which also contains svg.

```
{
	"type": "URL",
	"fields": {
		"type": {
			"url": "https://bitmapbytes.com",
		},
	},
	"dataString": "https://bitmapbytes.com",
	"ui": {
		"bgHexColor": "#ffffff",
		"qrHexColor": "#111111",
		"alignmentRadius": 0,
		"moduleRadius": 0,
		"moduleSeperation": 0,
	},
	"svg": "",
}
```

# Download

Open the components folder and there is a zipped "BB-QR-Field.zip" you can upload and install into your OCM Developer Components.

# Configure

1. Within OCM goto "Content" > "Asset Types"
2. Drop Data type of "Large Text"
3. In the data types appearance setting select BB-QR-Field.

That's it you can now create QR codes from URLs... 


# Coming soon...

- Choose Colours background and QR Code
- Add logo
- More types... Currently only URL works.

# How to build and deploy.

```
npm run build
```

- Once you build a dist folder is generated.
- Rename ocm-qr-field.umd.cjs to ocm-qr-field.umd.js
- Open up the component assets folder and drop this in and replace
- Redeploy to OCM.

# OCM Example QR Generator Field Component
![OCM_createQR](https://user-images.githubusercontent.com/364208/191046204-ea19e3cb-fd0a-4e47-85ca-e8a44d9dd956.png)
