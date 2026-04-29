```
let trustedURL;
if (window.trustedTypes && trustedTypes.createPolicy) {
	// Create a trusted policy for the script URL
	const policy = trustedTypes.createPolicy('myPolicy', {
		createScriptURL: (input) => input
	});
	trustedURL = policy.createScriptURL('https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.3.2/jspdf.min.js');
} else {
	console.warn("Trusted Types are not supported in this browser. Falling back to using a plain URL.");
	trustedURL = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.3.2/jspdf.min.js';
}

let jspdf = document.createElement("script");
jspdf.onload = function () {
	let pdf = new jsPDF();
	
	// 'blob:'から始まる画像だけを配列として正確に取得（不要なループを防ぐ）
	let blobImages = Array.from(document.querySelectorAll('img[src^="blob:"]'));

	blobImages.forEach((img, index) => {
		let canvasElement = document.createElement('canvas');
		let con = canvasElement.getContext("2d");
		
		// 【解像度の修正】表示サイズではなく、元画像の本来の解像度を使用
		canvasElement.width = img.naturalWidth;
		canvasElement.height = img.naturalHeight;
		
		// 背景を白で塗りつぶす（透過画像が黒くなるのを防ぐ）
		con.fillStyle = "#ffffff";
		con.fillRect(0, 0, canvasElement.width, canvasElement.height);
		con.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight);
		
		// 高画質(1.0)でJPEGデータ化
		let imgData = canvasElement.toDataURL("image/jpeg", 1.0);

		// PDFのページサイズ(A4)を取得 (jsPDF 1.3.2対応の記述)
		let pdfWidth = pdf.internal.pageSize.width || pdf.internal.pageSize.getWidth();
		let pdfHeight = pdf.internal.pageSize.height || pdf.internal.pageSize.getHeight();

		// 画像のアスペクト比を保ったままPDFに収める計算
		let imgRatio = img.naturalWidth / img.naturalHeight;
		let pdfRatio = pdfWidth / pdfHeight;
		let finalWidth, finalHeight;

		if (imgRatio > pdfRatio) {
			// 画像が横長の場合
			finalWidth = pdfWidth;
			finalHeight = pdfWidth / imgRatio;
		} else {
			// 画像が縦長の場合
			finalHeight = pdfHeight;
			finalWidth = pdfHeight * imgRatio;
		}

		// 画像をページ中央に配置
		let x = (pdfWidth - finalWidth) / 2;
		let y = (pdfHeight - finalHeight) / 2;

		// 座標とサイズを指定して画像をPDFへ追加
		pdf.addImage(imgData, 'JPEG', x, y, finalWidth, finalHeight);

		// 【白紙ページの修正】最後の画像の処理時以外のみ改ページを行う
		if (index < blobImages.length - 1) {
			pdf.addPage();
		}
	});

	if (blobImages.length > 0) {
		pdf.save("download.pdf");
	} else {
		console.warn("PDF化する blob 画像が見つかりませんでした。");
	}
};

// Use the trusted URL as the script source
jspdf.src = trustedURL;
document.body.appendChild(jspdf);
```
