<svelte:options tag="ocm-qr-field" />

<script>
	//svelte
	import { onMount, tick } from "svelte";

	//libs
	import qrGen from "./lib/utils/QRT.js";

	//Global
	let isMounted = false;
	let componentHeight;

	//bind dom
	let qrWrapp;

	//fieldvalue
	let dataset = {
		type: "URL",
		fields: {
			type: {
				url: "https://bitmapbytes.com",
			},
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

	//tmp
	let hex;

	//supported qr types
	let types = [
		"URL",
		"VCARD",
		//"TXT",
		//"EMAIL",
		//"SMS",
		//"WIFI",
		//"CRYPTO",
		//"TWITTER",
		//"FACEBOOK",
		//"PDF",
		//"MP3",
		//"APP STORE",
		//"IMAGES",
	];

	//Field
	let fieldSDK;
	let field;
	let currentLocale;

	//type[URL]
	let URL = "";

	//QRCode
	let dataString = '';
	let QRCode = "<div>Empty</div>";
	let bgHexColor = "#ffffff";
	let qrHexColor = "#111111";
	let alignmentRadius = 0;
	let moduleRadius = 0;
	let moduleSeperation = 0;

	//timer
	let QRGeneratorTimer

	//downloadQRURL
	let downloadUrl;

	//debug data
	let debug = '';

	//refresh QR;
	$: QRGenerator(
		dataString,
		bgHexColor,
		qrHexColor,
		alignmentRadius,
		moduleRadius,
		moduleSeperation
	);
	//update datastring
	$: if (URL) {
		dataset.fields.type.url = URL;
		dataString = URL;
	}

	$: if (componentHeight > 0) {
		setFrameHeight();
	}
	/**
	 *
	 */
	onMount(async () => {
		// this is the entry point to initialize the form sdk.
		if (window && window.editorSDK) {
			console.log("[Field Editor SDK]", window.editorSDK);
			await window.editorSDK.initSDK(initEditor);
		}
	});

	/**
	 * safeJsonParse
	*/
	function safeJsonParse(str) {
		try {
			return [null, JSON.parse(str)];
		} catch (err) {
			return [err];
		}
	}

	/**
	 *
	 */
	function initEditor(sdk) {
		console.log("[initEditor]");
		//sometimes double call is triggered using this hack
		if (isMounted) {
			return;
		}

		//define global
		fieldSDK = sdk;

		// retrieve the field object rendered by this custom editor
		field = fieldSDK.getField();

		// get the locale the form is currently in
		currentLocale = fieldSDK.getLocale();

		// register a custom validation function for this editor
		fieldSDK.setValidation(validateValue);

		//register render disabled function for this editor
		fieldSDK.registerDisable(renderDisabled);

		// retrieve the current field value
		const getFieldValue = field.getValue();
		const [err, result] = safeJsonParse(getFieldValue);
		//no data define set field with default structure/obj value
		if (err) {
			console.log('Failed to parse JSON: ',err.message,getFieldValue);
			setFieldValue();
		} else {
			console.log(result);
			dataset = JSON.parse(getFieldValue);
		}

		//grabs values stored and plces them into fields
		updateDataset();

		// if the field is updated externally, keep the editor in sync
		field.on("update", (value) => {
			console.log('[field][qr][update]');
			const [err, result] = safeJsonParse(value);
			//no data define set field with default structure/obj value
			if (err) {
				console.log('Failed to parse JSON: ',err.message,value);
				//reset value
				setFieldValue();
			} else {
				console.log(result);
				//push update
				dataset = JSON.parse(value);
			}
		});

		isMounted = true;
	}

	/**
	 * updateDataset
	*/
	function updateDataset() {

		//Update dataString based on type selection and any field binding values
		switch(dataset.type) {
			case 'URL':
				//update QR string
				dataString = dataset.fields.type.url; //auto updates dataString
				//update binding
				URL = dataset.fields.type.url;
			break;
		}

		//update QR UI
		QRCode = dataset.ui.QRCode;
		bgHexColor = dataset.ui.bgHexColor;
		qrHexColor = dataset.ui.qrHexColor;
		alignmentRadius = dataset.ui.alignmentRadius;
		moduleRadius = dataset.ui.moduleRadius;
		moduleSeperation = dataset.ui.moduleSeperation;
	}

	/**
	 * QRGenerator
	 * @param txt
	 * @param bgHexColor
	 * @param qrHexColor
	 * @param alignmentRadius
	 * @param moduleRadius
	 * @param moduleSeperation
	 */
	function QRGenerator(
		txt,
		bgHexColor,
		qrHexColor,
		alignmentRadius,
		moduleRadius,
		moduleSeperation
	) {
		clearTimeout(QRGeneratorTimer);

		//console.log('[QRGenerator]',txt)
		if (moduleSeperation > 0 && moduleRadius === 0) {
			moduleRadius = 1;
		}

		//console.log('[QRGenerator]',txt)
		if (moduleRadius > 0 && alignmentRadius === 0) {
			alignmentRadius = 1;
		}

		//grab svg generated
		const myQR = qrGen(
			txt,
			bgHexColor,
			qrHexColor,
			alignmentRadius,
			moduleRadius,
			moduleSeperation
		);

		//update fieldset data
		dataset.dataString = txt;
		//update ui
		dataset.ui = {
			bgHexColor,
			qrHexColor,
			alignmentRadius,
			moduleRadius,
			moduleSeperation,
		}
		//update svg val
		dataset.svg = myQR;

		//update svg
		QRCode = myQR;

		//generate download link
		QRGeneratorTimer = setTimeout(updateDownloadLink,300);
		
	}

	/**
	 * setFieldValue
	 */
	function setFieldValue() {
		console.log('[setFieldValue]');
		//check field ok..
		if ((typeof(dataset) === 'object') && (dataset !== null)) {
			console.log('..settingvalue',dataset);
			
			let data = JSON.stringify(dataset);
			if (field) {
				field.setValue(data);
			}

			debug = data;
		} else {
			console.error('error: dataset type not an object....',dataset);
		}
	}

	/**
	 * validates whether entered number of characters is more than 20000
	 * @param value
	 */
	function validateValue(value) {
		//validate
		let isValid = true;

		const [err, result] = safeJsonParse(value);

		//no data define set field with default structure/obj value
		if (err) {
			return {
				isValid: false,
				title: 'QR Code Invalid',
				message: 'An Error occured with the QR Code generator unable to save to OCM'
			};
		}

		return isValid;
	}

	/**
	 * disable/enable call back that gets called if the content form
	 * needs to enable/disable this editor
	 * @param value
	 */
	function renderDisabled(value) {
		if (value) {
			//document.getElementById('textInput').disabled = true;
		} else {
			//document.getElementById('textInput').disabled = false;
		}
	}

	/**
	 * setFrameHeight
	*/
	function setFrameHeight() {
		//add delay to refresh heights
		if (fieldSDK) {
			setTimeout(() => {
				fieldSDK.resize({ 
					height: `${componentHeight+40}px`, 
				});
			},10);
		}
	}

	function updateDownloadLink() {
		//get svg element.
		if (qrWrapp) {

			const svg = qrWrapp.getElementsByTagName("svg")[0];
			console.log('updateDownloadLink',svg,qrWrapp.getElementsByTagName("svg"))
			if (svg) {
				//get svg source.
				const serializer = new XMLSerializer();
				let source = serializer.serializeToString(svg);

				//add name spaces.
				if(!source.match(/^<svg[^>]+xmlns="http\:\/\/www\.w3\.org\/2000\/svg"/)){
					source = source.replace(/^<svg/, '<svg xmlns="http://www.w3.org/2000/svg"');
				}
				if(!source.match(/^<svg[^>]+"http\:\/\/www\.w3\.org\/1999\/xlink"/)){
					source = source.replace(/^<svg/, '<svg xmlns:xlink="http://www.w3.org/1999/xlink"');
				}

				//add xml declaration
				source = '<?xml version="1.0" standalone="no"?>\r\n' + source;

				//convert svg source to URI data scheme.
				downloadUrl = "data:image/svg+xml;charset=utf-8,"+encodeURIComponent(source);
					
			}
		}
	}
</script>

<!-- Window -->
<svelte:window on:blur="{setFieldValue}"></svelte:window>
<!-- xWindow -->

<!-- Wrapper -->
<section class="bitmapbytes-QRFormField">
	<article bind:clientHeight="{componentHeight}">
		<!-- DEBUG -->
		<div style="display:none">
			<textarea>{debug}</textarea>
		</div>
		<!-- xDEBUG -->

		<!-- QR Type Selector-->
		<div>
			<dl>
				<dt style="text-align:center; margin-bottom:10px;">
					<label for="fQRType">QR Type</label>
				</dt>
				<dd>
					<ul>
						{#each types as type}
							<li
								on:click={async() => {
									dataset.type = type;
								}}
								class:active={type === dataset.type}
							>
								{type}
							</li>
						{/each}
					</ul>
				</dd>
			</dl>
		</div>
		<!-- xQR Type Selector-->

		<!-- Sepertor -->
		<div class="hr"><hr /></div>
		<!-- xSepertor -->

		<!-- QR Data -->
		<main style="padding:10px">
			{#if dataset.type === "URL"}
				<dl>
					<dt><label for="fURL">{dataset.type}</label></dt>
					<dd class="input">
						<input
							id="fURL"
							on:blur={setFieldValue}
							bind:value={URL}
							type="text"
							placeholder="Enter {dataset.type}"
						/>
					</dd>
				</dl>
			{/if}
			{#if dataset.type === "VCARD"}
			<p><b>.... Working on other QR TYPES... coming soon!</b></p>
				<div>
					<dl>
						<dt>
							<label for="fName">Your Name:</label>
						</dt>
						<dd class="split">
							<div class="input">
								<input
									id="fName"
									type="text"
									placeholder="First name"
									
								/>
							</div>
							<div class="input">
								<input
									id="lastname"
									type="text"
									placeholder="Last name"
									
								/>
							</div>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fContact">Contact:</label>
						</dt>
						<dd>
							<div class="input" style="margin-bottom:10px;">
								<input
									id="fContact"
									type="text"
									placeholder="Mobile"
									/>
							</div>
							<div class="split">
								<div class="input">
									<input
										type="text"
										placeholder="Phone"
										/>
								</div>
								<div class="input">
									<input
										type="text"
										placeholder="Fax"
										/>
								</div>
							</div>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fEmail">Email:</label>
						</dt>
						<dd class="input">
							<input
								id="fEmail"
								type="text"
								placeholder="your@email.com"
								
							/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fCompany">Company:</label>
						</dt>
						<dd class="split">
							<div class="input">
								<input
									id="fCompany"
									type="text"
									placeholder="Company"
									/>
							</div>
							<div class="input">
								<input
									type="text"
									placeholder="Your Job"
									/>
							</div>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fStreet">Street:</label>
						</dt>
						<dd class="input">
							<input
								id="fStreet"
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fCity">City:</label>
						</dt>
						<dd class="split">
							<div class="input">
								<input
									id="fCity"
									type="text"
									/>
							</div>
							<div class="input">
								<input
									type="text"
									placeholder="ZIP"
									/>
							</div>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fState">State:</label>
						</dt>
						<dd class="input">
							<input
								id="StafStatete"
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fCountry">Country:</label>
						</dt>
						<dd class="input">
							<input
								id="fCountry"
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label for="fWebsite">Website:</label>
						</dt>
						<dd class="input">
							<input
								id="fWebsite"
								type="text"
								placeholder="www.your-website.com"
								/>
						</dd>
					</dl>
				</div>
			{/if}
			<!--{#if dataset.type === "TXT"}
				<dl>
					<dt><label for="fTxt">{dataset.type}</label></dt>
					<dd class="input">
						<input
							id="fTxt"
							type="text"
							placeholder="Enter {dataset.type}"
						/>
					</dd>
				</dl>
			{/if}
			{#if dataset.type === "EMAIL"}
				<div>
					<dl>
						<dt>
							<label>Email:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Subject:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Message:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								/>
						</dd>
					</dl>
				</div>
			{/if}
			{#if dataset.type === "SMS"}
				<div>
					<dl>
						<dt>
							<label>Number:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Message:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								/>
						</dd>
					</dl>
				</div>
			{/if}
			{#if dataset.type === "WIFI"}
				<div>
					<dl>
						<dt>
							<label>Network Name:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								placeholder="SSID"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Password:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Encryption:</label>
						</dt>
						<dd class="input">
							<ul class="split">
								<li><span>None</span></li>
								<li><span>WPA/WPA2</span></li>
								<li><span>WEP</span></li>
							</ul>
						</dd>
					</dl>
				</div>
			{/if}
			{#if dataset.type === "CRYPTO"}
				<div>
					<dl>
						<dt>
							<label>Cryptocurrency:</label>
						</dt>
						<dd class="input">
							<ul class="split">
								<li><span>Bitcoin</span></li>
								<li><span>Bitcoin Cash</span></li>
								<li><span>Ether</span></li>
								<li><span>Litecoin</span></li>
								<li><span>Dash</span></li>
							</ul>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Amount:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Receiver:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								placeholder="Wallet Address"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Message:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								placeholder="Optional"
								/>
						</dd>
					</dl>
				</div>
			{/if}
			{#if dataset.type === "TWITTER"}
				<div>
					<dl>
						<dt>
							<label>Choose an option:</label>
						</dt>
						<dd class="input">
							<ul class="split">
								<li><span>Link to your profile</span></li>
								<li><span>Post a tweet</span></li>
							</ul>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Your username:</label>
						</dt>
						<dd class="input">
							<input
								type="text"
								placeholder="@username"
								/>
						</dd>
					</dl>
				</div>
			{/if}-->
		</main>
		<!-- QR Data -->
		
		<!-- Seperator -->
		<div class="hr"><hr /></div>
		<!-- Seperator -->
		
		<!-- Settings -->
		<div class="split settings">
			<div>
				<dl>
					<dt><label for="fAlignmentRad">Alignment Radius</label></dt>
					<dd class="input range">
						<input
							type="range"
							id="fAlignmentRad"
							name="fAlignmentRad"
							min="0"
							max="100"
							bind:value={alignmentRadius}
						/>
					</dd>
				</dl>
				<dl>
					<dt><label for="fModuleRad">Module Radius</label></dt>
					<dd class="input range">
						<input
							type="range"
							id="fModuleRad"
							name="fModuleRad"
							min="0"
							max="100"
							bind:value={moduleRadius}
						/>
					</dd>
				</dl>
				<dl>
					<dt><label for="fModuleSep">Module Seperation</label></dt>
					<dd class="input range">
						<input
							type="range"
							id="fModuleSep"
							name="fModuleSep"
							min="0"
							max="40"
							bind:value={moduleSeperation}
						/>
					</dd>
				</dl>
				<!--
				<dl>
					<dt><label>Background Colour</label></dt>
					<dd class="input range">
					</dd>
				</dl>
				<dl>
					<dt><label>QR Colour</label></dt>
					<dd class="input range">
					</dd>
				</dl>-->

			</div>
			<div class="preview">
				<div class="qrWrapp" bind:this="{qrWrapp}">
					{@html QRCode}
					{#if (downloadUrl)}
					<a class="download" href="{downloadUrl}" download="QRCode.svg">Download SVG</a>
					{/if}
				</div>
			</div>
		</div>
		<!-- Settings -->
	</article>
</section>

<!-- xWrapper -->
<style>
	section {
	}
	main {
		max-height:400px;
		overflow-y: auto;
		overflow-x: hidden;
	}
	.split.settings > div {
		padding:10px;
	}
	article {
		padding: 10px;
		overflow: hidden;
		background: #fff;
		border-radius: 10px;
		box-shadow: 0px 0px 0px 0px rgb(255, 255, 255),
			0px 0px 0px 1px rgba(51, 65, 85, 0.1),
			0px 20px 25px -5px rgba(0, 0, 0, 0.05),
			0px 8px 10px -6px rgba(0, 0, 0, 0.05);
		/*width: 400px;*/
	}
	ul {
		display: flex;
		flex-wrap: wrap;
		margin: 0px;
		padding: 0px;
		gap: 10px;
		justify-content: center;
	}
	li {
		list-style: none;
		width: 80px;
		height: 50px;
		box-shadow: 0px 0px 0px 0px rgb(255, 255, 255),
			0px 0px 0px 1px rgba(51, 65, 85, 0.1),
			0px 1px 2px 0px rgba(0, 0, 0, 0.05);
		border-radius: 4px;
		font-weight: bold;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: 0.675em;
		cursor: pointer;
		text-transform: uppercase;
		color: rgba(51, 65, 85, 0.8);
		transition: background 0.3s, box-shadow 0.3s;
	}

	li.active,
	li:hover {
		box-shadow: 0px 0px 0px 0px rgb(255, 255, 255),
			0px 0px 0px 1px rgba(51, 65, 85, 0.1),
			0px 1px 2px 0px rgba(0, 0, 0, 0.05),
			inset 0px 1px 1px 0px rgba(0, 0, 0, 0.5);
		background: rgba(51, 65, 85, 0.1);
		color: #000;
	}
	label {
		font-size: 0.875em;
	}

	.split {
		display: flex;
		gap: 10px;
	}

	.split > div {
		flex: 1;
	}

	.input {
		display: flex;
		box-shadow: 0px 0px 0px 0px rgb(255, 255, 255),
			0px 0px 0px 1px rgba(51, 65, 85, 0.1),
			0px 1px 2px 0px rgba(0, 0, 0, 0.05);
		border-radius: 4px;
	}
	.input.range {
		border-radius: 100px;
	}
	input {
		flex: 1;
		border: 0px;
		overflow: hidden;
	}
	input[type="text"] {
		min-height: 30px;
		padding: 0px 10px;
		background: transparent;
	}

	dd {
		margin: 0px;
		padding: 0px;
	}
	label {
		font-weight: bold;
	}

	input[type="range"] {
		margin: 6px 6px;
	}

	.hr hr {
		display: none;
	}
	.hr {
		height: 8px;
		background: rgb(241 245 249);
		border-radius: 100px;
	}

	.qrWrapp {
		position: relative;
		flex: 1;
		box-shadow: 0px 0px 0px 0px rgb(255, 255, 255),
			0px 0px 0px 1px rgba(51, 65, 85, 0.1),
			0px 1px 2px 0px rgba(0, 0, 0, 0.05);
		border-radius: 10px;
		overflow: hidden;
	}
	.preview {
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.bitmapbytes-QRFormField ::-webkit-scrollbar {
		display:block;
		width:8px;
	}
	
	.bitmapbytes-QRFormField ::-webkit-scrollbar-thumb {
		background: #7C8B9A; 
		border-radius: 100px;
	}

	.bitmapbytes-QRFormField ::-webkit-scrollbar-button {
		width:8px;
		height:10px;
	}
	.download {
		background: rgb(15,23,42);
		border-radius: 8px;
		padding:4px 8px;
		margin:0px auto 10px;
		max-width:140px;
		display: block;
		text-align:center;
		color:#fff;
	}
</style>
