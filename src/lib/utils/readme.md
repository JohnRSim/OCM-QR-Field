# utils

## QRCode Generator
I'm using https://github.com/aruZeta/QRgen 

### Features

- Supports all QR versions: from 1 to 40.
- Supports all EC (Error Correction) levels: L, M, Q and H.
- Supports numeric mode, alphanumeric mode and byte mode.
- Supports printing a QR code on your terminal via standard output.
- Supports printing a QR code to SVG, with custom colors, using circles, etc.

## Compiling latest release
Download install Nim - nim-lang.org


## Create placeholder "QRT.nim" to expose functions


```
import
  QRgen,
  std/[jsconsole],
  std/[dom]

block:
  console.log(newQR);
  let qr = newQR("https://bitmapbytes.com")
  qr.printTerminal
```

#### Compile
```
nim js -d:release "QRT.nim"
```

Generates JS Lib in build folder.

### Update QRT.js in this folder

### Manually change build code

```
//comment oout label1: block and update vars as below
//manually map to generated functions
const getMostEfficientMode = getMostEfficientMode_788529169;
const newQR = newQR_620756998;
const getSmallestVersion = getSmallestVersion_788529195;
const printSvg = printSvg_989855780;

function qrGen(txt,bgHexColor,qrHexColor,alignmentRadius,moduleRadius,moduleSeperation) {
	//check string
	if ((txt) && (txt.length === 0)) {
		console.error('Please supply QR String');
		return;
	}

	//define globals
	bgHexColor = bgHexColor || '#ffffff';
	qrHexColor = qrHexColor || '#000000';
	alignmentRadius = alignmentRadius || 0;
	moduleRadius = moduleRadius || 0;
	moduleSeperation = moduleSeperation || 0;
	
	//init
	const mkstring = makeNimstrLit(txt);
	const efficientMode = getMostEfficientMode(mkstring);
	const qr = nimCopy(null, newQR(mkstring, 0, efficientMode, getSmallestVersion(mkstring, efficientMode, 0)), NTI687865860);
	
	//return SVG
	return toJSStr(printSvg(qr, makeNimstrLit(bgHexColor), makeNimstrLit(qrHexColor), alignmentRadius, moduleRadius, moduleSeperation, makeNimstrLit("qrCode"), [], false));
}

//expose for window/import
window.qrGen = qrGen;
export default qrGen;
```