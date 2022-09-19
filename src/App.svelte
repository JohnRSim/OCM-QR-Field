<svelte:options tag="ocm-qr-field" />

<script>
	//svelte
	import { onMount } from 'svelte';
	import qrGen from './lib/utils/QRT.js';
	
	//Global
	let isMounted = false;

	//Field
	let fieldSDK;
	let field;
	let fieldValue ='';

	//QRCode
	let QRCode = '<div>Empty</div>';
	let bgHexColor;
	let qrHexColor;
	let alignmentRadius;
	let moduleRadius;
	let moduleSeperation;

	//refresh QR;
	$: QRGenerator(fieldValue,bgHexColor,qrHexColor,alignmentRadius,moduleRadius,moduleSeperation)

	//Tmp
	let currentLocale;
	let localeStrings = {
		en: {
			validation: {
				title: 'Invalid value',
				message: 'You have entered more than 20000 characters'
			}
		},
			es: {
			validation: {
				title: 'valor no válido',
				message: 'Has introducido más de 20000 caracteres'
			}
		},
		fr: {
			validation: {
				title: 'valeur invalide',
				message: 'Vous avez entré plus de 20000 caractères'
			}
		}
	};

	/**
	 * 
	*/
	onMount(async () => {
		// this is the entry point to initialize the form sdk.
		if (window.editorSDK) {
			console.log('[Content Form SDK]',window.contentFormSDK);
			await window.editorSDK.initSDK(initEditor);
		}
		//QRGenerator('s');
	});
		
	/**
	 * 
	*/
	function initEditor(sdk) {
		console.log('[initEditor]');
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
		field.on('update', (value) => {
			fieldValue = value ? value : '';
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
	function QRGenerator(txt,bgHexColor,qrHexColor,alignmentRadius,moduleRadius,moduleSeperation) {
		//console.log('[QRGenerator]',txt)
		const myQR = qrGen(txt);
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
				title: getTranslatedString('validation.title', currentLocale),
				message: getTranslatedString('validation.message', currentLocale)
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
		key = key || '';
		locale = locale || 'en';

		var propList = key ? key.split('.') : [],
			currValue = localeStrings[locale];

		for (var i = 0, len = propList.length; i < len; i++) {
		currValue[propList[i]] = currValue[propList[i]] || {};
		currValue = currValue[propList[i]];
		}

		var translatedString = (typeof currValue === 'string') ? currValue : undefined;

		return translatedString;
	}
</script>


<!-- Wrapper -->
<section class="bitmapbytes-QRFormField">
	{@html QRCode}
	<input on:blur="{setFieldValue}" bind:value="{fieldValue}" />
</section>
<!-- xWrapper -->

<style>
	

</style>
