<svelte:options tag="ocm-qr-field" />

<script>
	//svelte
	import { onMount } from "svelte";
	import qrGen from "./lib/utils/QRT.js";

	//Global
	let isMounted = false;

	let dataset = {
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
	//supported qr types
	let types = [
		"URL",
		"VCARD",
		"TXT",
		"EMAIL",
		"SMS",
		"WIFI",
		"CRYPTO",
		"TWITTER",
		//"FACEBOOK",
		//"PDF",
		//"MP3",
		//"APP STORE",
		//"IMAGES",
	];

	//Field
	let fieldSDK;
	let field;
	let fieldValue = "";

	//QRCode
	let QRCode = "<div>Empty</div>";
	let bgHexColor = "#ffffff";
	let qrHexColor = "#111111";
	let alignmentRadius = 0;
	let moduleRadius = 0;
	let moduleSeperation = 0;

	//refresh QR;
	$: QRGenerator(
		fieldValue,
		bgHexColor,
		qrHexColor,
		alignmentRadius,
		moduleRadius,
		moduleSeperation
	);

	//Tmp
	let currentLocale;
	let localeStrings = {
		en: {
			validation: {
				title: "Invalid value",
				message: "You have entered more than 20000 characters",
			},
		},
		es: {
			validation: {
				title: "valor no válido",
				message: "Has introducido más de 20000 caracteres",
			},
		},
		fr: {
			validation: {
				title: "valeur invalide",
				message: "Vous avez entré plus de 20000 caractères",
			},
		},
	};

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
		fieldValue = field.getValue();

		// if the field is updated externally, keep the editor in sync
		field.on("update", (value) => {
			fieldValue = value ? value : "";
		});

		isMounted = true;
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
		//console.log('[QRGenerator]',txt)
		if (moduleRadius > 0 && alignmentRadius === 0) {
			alignmentRadius = 1;
		}
		const myQR = qrGen(
			txt,
			bgHexColor,
			qrHexColor,
			alignmentRadius,
			moduleRadius,
			moduleSeperation
		);
		QRCode = myQR;
	}

	/**
	 * setFieldValue
	 */
	function setFieldValue() {
		field.setValue(fieldValue);
	}

	/**
	 * validates whether entered number of characters is more than 20000
	 * @param value
	 */
	function validateValue(value) {
		var isValid = true;

		if (value && value.length > 20000) {
			isValid = false;

			return {
				isValid: false,
				title: getTranslatedString("validation.title", currentLocale),
				message: getTranslatedString(
					"validation.message",
					currentLocale
				),
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
	 * getTranslatedString
	 * @param key
	 * @param locale
	 */
	function getTranslatedString(key, locale) {
		key = key || "";
		locale = locale || "en";

		var propList = key ? key.split(".") : [],
			currValue = localeStrings[locale];

		for (var i = 0, len = propList.length; i < len; i++) {
			currValue[propList[i]] = currValue[propList[i]] || {};
			currValue = currValue[propList[i]];
		}

		var translatedString =
			typeof currValue === "string" ? currValue : undefined;

		return translatedString;
	}
</script>

<!-- Wrapper -->
<section class="bitmapbytes-QRFormField">
	<article>
		<div>
			<dl>
				<dt style="text-align:center; margin-bottom:10px;">
					<label>QR Type</label>
				</dt>
				<dd>
					<ul>
						{#each types as type}
							<li
								on:click={() => {
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
		<div class="hr"><hr /></div>
		<main style="padding:10px">
			{#if dataset.type === "URL"}
				<dl>
					<dt><label>{dataset.type}</label></dt>
					<dd class="input">
						<input
							on:blur={setFieldValue}
							bind:value={fieldValue}
							type="text"
							placeholder="Enter {dataset.type}"
						/>
					</dd>
				</dl>
			{/if}
			{#if dataset.type === "VCARD"}
				<div>
					<dl>
						<dt>
							<label>Your Name:</label>
						</dt>
						<dd class="split">
							<div class="input">
								<input
									id="firstname"
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
							<label>Contact:</label>
						</dt>
						<dd>
							<div class="input" style="margin-bottom:10px;">
								<input
									id="mobileNumber"
									type="text"
									placeholder="Mobile"
									/>
							</div>
							<div class="split">
								<div class="input">
									<input
										id="phoneNumber"
										type="text"
										placeholder="Phone"
										/>
								</div>
								<div class="input">
									<input
										id="faxNumber"
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
							<label>Email:</label>
						</dt>
						<dd class="input">
							<input
								id="email"
								type="text"
								placeholder="your@email.com"
								
							/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Company:</label>
						</dt>
						<dd class="split">
							<div class="input">
								<input
									id="company"
									type="text"
									placeholder="Company"
									/>
							</div>
							<div class="input">
								<input
									id="job"
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
							<label>Street:</label>
						</dt>
						<dd class="input">
							<input
								id="street"
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>City:</label>
						</dt>
						<dd class="split">
							<div class="input">
								<input
									id="city"
									type="text"
									/>
							</div>
							<div class="input">
								<input
									id="zip"
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
							<label>State:</label>
						</dt>
						<dd class="input">
							<input
								id="State"
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Country:</label>
						</dt>
						<dd class="input">
							<input
								id="Country"
								type="text"
								/>
						</dd>
					</dl>
				</div>
				<div>
					<dl>
						<dt>
							<label>Website:</label>
						</dt>
						<dd class="input">
							<input
								id="Website"
								type="text"
								placeholder="www.your-website.com"
								/>
						</dd>
					</dl>
				</div>
			{/if}
			{#if dataset.type === "TXT"}
				<dl>
					<dt><label>{dataset.type}</label></dt>
					<dd class="input">
						<input
							on:blur={setFieldValue}
							bind:value={fieldValue}
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
			{/if}
		</main>
		<div class="hr"><hr /></div>
		<div class="split settings">
			<div>
				<dl>
					<dt><label>Alignment Radius</label></dt>
					<dd class="input range">
						<input
							type="range"
							id="volume"
							name="volume"
							min="0"
							max="100"
							bind:value={alignmentRadius}
						/>
					</dd>
				</dl>
				<dl>
					<dt><label>Module Radius</label></dt>
					<dd class="input range">
						<input
							type="range"
							id="volume"
							name="volume"
							min="0"
							max="100"
							bind:value={moduleRadius}
						/>
					</dd>
				</dl>
				<dl>
					<dt><label>Module Seperation</label></dt>
					<dd class="input range">
						<input
							type="range"
							id="volume"
							name="volume"
							min="0"
							max="60"
							bind:value={moduleSeperation}
						/>
					</dd>
				</dl>
			</div>
			<div class="preview">
				<div class="qrWrapp">
					{@html QRCode}
				</div>
			</div>
		</div>
	</article>
</section>

<!-- xWrapper -->
<style>
	section {
		height:940px
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
		width: 400px;
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
		flex: 1;
		box-shadow: 0px 0px 0px 0px rgb(255, 255, 255),
			0px 0px 0px 1px rgba(51, 65, 85, 0.1),
			0px 1px 2px 0px rgba(0, 0, 0, 0.05);
		border-radius: 4px;
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
</style>
