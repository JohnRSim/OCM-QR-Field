![QR-Generator](https://user-images.githubusercontent.com/364208/190979725-0c1e4641-80bb-4503-b4e5-6d5070bae4ea.png)


# OCM-QR-Field (In Development)

Free Custom Component built with Svelte for generating QR Codes from form fields; installed as a component and used directly within an OCM Form.



https://user-images.githubusercontent.com/364208/191071289-0b733cf5-fcfc-46fd-8c4c-feae37817c34.mp4



# Data Type "Large Text"
This field stores json data which also contains svg.

```
{
		type: "URL",
		fields: {
			url: "https://bitmapbytes.com",
		},
		dataString: "https://bitmapbytes.com",
		ui: {
			bgHexColor: "#ffffff",
			qrHexColor: "#111111",
			alignmentRadius: 0,
			moduleRadius: 0,
			moduleSeperation: 0,
		},
		svg: "",
	};
```

# OCM Example QR Generator Field Component
![OCM_createQR](https://user-images.githubusercontent.com/364208/191046204-ea19e3cb-fd0a-4e47-85ca-e8a44d9dd956.png)
