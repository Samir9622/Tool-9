<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Image Compression Tool</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div class="home-container">
    <h1>Welcome to Image Compression Tool</h1>
    <a href="compressor.html" class="tool-button">Open Tool</a>
  </div>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Compressor</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>Image Compression Tool</h1>
    <a href="index.html" class="back-button">← Back</a>
  </header>

  <div class="upload-area" id="drop-zone">
    <p>Drag & Drop Images Here or Click to Upload</p>
    <input type="file" id="file-input" multiple accept="image/*">
  </div>

  <div class="settings">
    <label for="quality">Compression Quality: <span id="quality-value">80</span>%</label>
    <input type="range" id="quality" min="10" max="100" value="80">
    
    <label class="switch">
      <input type="checkbox" id="webp-toggle">
      <span class="slider round"></span>
    </label>
    <span>Convert to WebP</span>
  </div>

  <div class="preview" id="preview"></div>

  <div class="actions">
    <button id="download-all">Download All</button>
  </div>

  <script src="script.js"></script>
</body>
</html>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background: #f5f5f5;
  color: #333;
  text-align: center;
}

header {
  background: #333;
  color: #fff;
  padding: 15px;
  position: relative;
}

.back-button {
  position: absolute;
  left: 15px;
  top: 15px;
  color: white;
  text-decoration: none;
}

.home-container {
  margin-top: 100px;
}

.tool-button {
  padding: 12px 24px;
  background-color: #008CBA;
  color: white;
  text-decoration: none;
  font-size: 18px;
  border-radius: 5px;
}

.upload-area {
  border: 2px dashed #999;
  padding: 40px;
  margin: 30px;
  background: white;
  cursor: pointer;
}

#file-input {
  display: none;
}

.settings {
  margin: 20px;
}

.preview {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 15px;
  margin: 20px;
}

.preview img {
  max-width: 200px;
  max-height: 200px;
  object-fit: contain;
  border: 1px solid #ccc;
  padding: 5px;
  background: white;
}

.actions {
  margin: 20px;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  background: #28a745;
  color: white;
  border: none;
  cursor: pointer;
}

/* Toggle Switch */
.switch {
  position: relative;
  display: inline-block;
  width: 50px;
  height: 24px;
  margin: 0 10px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 24px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .slider {
  background-color: #4CAF50;
}

input:checked + .slider:before {
  transform: translateX(26px);
}
const dropZone = document.getElementById("drop-zone");
const fileInput = document.getElementById("file-input");
const qualitySlider = document.getElementById("quality");
const qualityValue = document.getElementById("quality-value");
const preview = document.getElementById("preview");
const webpToggle = document.getElementById("webp-toggle");
const downloadAllBtn = document.getElementById("download-all");

let compressedImages = [];

qualitySlider.addEventListener("input", () => {
  qualityValue.textContent = qualitySlider.value;
});

dropZone.addEventListener("click", () => fileInput.click());
dropZone.addEventListener("dragover", (e) => {
  e.preventDefault();
  dropZone.style.backgroundColor = "#eee";
});
dropZone.addEventListener("dragleave", () => {
  dropZone.style.backgroundColor = "white";
});
dropZone.addEventListener("drop", (e) => {
  e.preventDefault();
  dropZone.style.backgroundColor = "white";
  handleFiles(e.dataTransfer.files);
});

fileInput.addEventListener("change", () => {
  handleFiles(fileInput.files);
});

function handleFiles(files) {
  Array.from(files).forEach(file => {
    if (!file.type.startsWith("image/")) return;

    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => compressAndPreview(img, file.name);
      img.src = e.target.result;
    };
    reader.readAsDataURL(file);
  });
}

function compressAndPreview(img, originalName) {
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");

  const maxW = 800;
  const scale = Math.min(1, maxW / img.width);
  canvas.width = img.width * scale;
  canvas.height = img.height * scale;
  ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

  const useWebP = webpToggle.checked;
  const type = useWebP ? "image/webp" : "image/jpeg";
  const ext = useWebP ? ".webp" : ".jpg";
  const quality = parseInt(qualitySlider.value) / 100;

  canvas.toBlob(blob => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = originalName.replace(/\.[^/.]+$/, "") + ext;

    const imgEl = document.createElement("img");
    imgEl.src = url;

    const wrapper = document.createElement("div");
    wrapper.appendChild(imgEl);
    wrapper.appendChild(a);
    a.textContent = "Download";

    preview.appendChild(wrapper);
    compressedImages.push({ url, filename: a.download });
  }, type, quality);
}

downloadAllBtn.addEventListener("click", () => {
  compressedImages.forEach(file => {
    const a = document.createElement("a");
    a.href = file.url;
    a.download = file.filename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  });
});
