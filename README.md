<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Secrets Manager</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
  <style>
    /* General Reset */
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: Arial, sans-serif;
    }

    body {
      background-color: #f4f4f9;
      color: #333;
      line-height: 1.6;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 20px;
    }

    /* Container */
    .container {
      max-width: 600px;
      width: 100%;
      background: #fff;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }

    h2, h3 {
      margin-bottom: 16px;
      color: #333;
      font-weight: bold;
    }

    /* Input and Button Styling */
    label {
      display: block;
      font-weight: bold;
      margin-bottom: 5px;
      color: #555;
    }

    input[type="text"],
    input[type="password"],
    select {
      width: 100%;
      padding: 10px;
      margin-bottom: 15px;
      border: 1px solid #ddd;
      border-radius: 4px;
      background-color: #fafafa;
    }

    input[type="file"] {
      display: block;
      margin-bottom: 15px;
    }

    .input-wrapper {
      position: relative;
    }

    .icon1 {
      position: absolute;
      right: 30px;
      top: 50%;
      transform: translateY(-50%);
      cursor: pointer;
    }
    .icon {
      position: absolute;
      right: 10px;
      top: 50%;
      transform: translateY(-50%);
      cursor: pointer;
    }

    button {
      background-color: #5c67f2;
      color: #fff;
      padding: 10px 15px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      margin-top: 10px;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: #3c4cb2;
    }

    /* List Styling */
    #vendorList {
      list-style: none;
      padding-left: 0;
      margin-top: 15px;
      margin-bottom: 20px;
    }

    #vendorList li {
      background: #f9f9f9;
      padding: 8px;
      margin-bottom: 8px;
      border: 1px solid #ddd;
      border-radius: 4px;
      font-size: 0.9em;
    }

    /* Decrypted Value Display */
    #decryptedValue {
      font-weight: bold;
      color: #5c67f2;
      font-size: 1.1em;
    }

    /* Section Dividers */
    hr {
      margin: 20px 0;
      border: none;
      border-top: 1px solid #eee;
    }
  </style>
</head>
<body>

<div class="container">
  <h2>Manage Secrets with Encrypted Values</h2>

  <div>
    <label for="fileInput">Load CSV File:</label>
    <input type="file" id="fileInput" accept=".csv">
  </div>

  <h3>Add or Update Business</h3>
  <div>
    <label for="vendorId">Business Name:</label>
    <input type="text" id="vendorId" placeholder="Enter Business/Site Name">
  </div>
  <div class="input-wrapper">
    <label for="secretKey">Secret Key:</label>
    <input type="password" id="secretKey" placeholder="Enter Secret Key">
    <span class="icon" onclick="toggleVisibility('secretKey')">👁️</span>
  </div>
  <div class="input-wrapper">
    <label for="value">Value:</label>
    <input type="password" id="value" placeholder="Enter Value to Encrypt">
    <span class="icon" onclick="toggleVisibility('value')">👁️</span>
  </div>
  <div>
    <button onclick="addOrUpdateVendor()">Add/Update Business</button>
  </div>

  <div>
    <button onclick="saveToFile()">Save to CSV</button>
  </div>

  <h3>Vendors</h3>
  <ul id="vendorList"></ul>

  <hr>

  <h3>Decrypt Vendor Value</h3>
  <div>
    <label for="vendorSelect">Select a Business:</label>
    <select id="vendorSelect" onchange="clearDecryptedValue()">
      <option value="">--Select a Business--</option>
    </select>
  </div>
  <div class="input-wrapper">
    <label for="decryptSecretKey">Enter Secret Key:</label>
    <input type="password" id="decryptSecretKey" placeholder="Enter Secret Key">
    <span class="icon" onclick="toggleVisibility('decryptSecretKey')">👁️</span>
  </div>
  <div>
    <button onclick="showDecryptedValue()">Show Decrypted Value</button>
  </div>
 
 <div class="input-wrapper">
    <label for="decryptedValue">Decrypted Value:</label>
    <input type="password" id="decryptedValue">
    <span class="icon" onclick="toggleVisibility('decryptedValue')">👁️</span>
    <span class="icon1"  onclick="copyToClipboard()">📋</span>

  </div>    
    
</div>

<script>
  let vendorData = [];

  function toggleVisibility(id) {
    const input = document.getElementById(id);
    input.type = input.type === 'password' ? 'text' : 'password';
  }

  document.getElementById("fileInput").addEventListener("change", function(event) {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = function(e) {
        const csvContent = e.target.result;
        parseCSV(csvContent);
        populateVendorSelect();
      };
      reader.readAsText(file);
    }
  });

  function parseCSV(csvContent) {
    vendorData = csvContent.split("\n").map(line => {
      const [vendorId, secretKey, encryptedValue] = line.split(",");
      return {
        vendorId: vendorId.trim(),
        secretKey: secretKey.trim(),
        encryptedValue: encryptedValue ? encryptedValue.trim() : ""
      };
    });
  }

  function displayVendors() {
    const vendorList = document.getElementById("vendorList");
    vendorList.innerHTML = "";
    vendorData.forEach(vendor => {
      const listItem = document.createElement("li");
      vendorList.appendChild(listItem);
    });
  }

  function addOrUpdateVendor() {
    const vendorId = document.getElementById("vendorId").value.trim();
    const secretKey = document.getElementById("secretKey").value.trim();
    const value = document.getElementById("value").value.trim();

    if (!vendorId || !secretKey || !value) {
      alert("Please enter Business ID, Secret Key, and Value.");
      return;
    }

    const encryptedValue = CryptoJS.AES.encrypt(value, secretKey).toString();

    const vendorIndex = vendorData.findIndex(vendor => vendor.vendorId === vendorId);
    if (vendorIndex >= 0) {
      vendorData[vendorIndex] = { vendorId, secretKey, encryptedValue };
      alert(`Updated Business name: ${vendorId}`);
    } else {
      vendorData.push({ vendorId, secretKey, encryptedValue });
      alert(`Added new Business name: ${vendorId}`);
    }

    
    populateVendorSelect();
  }

  function populateVendorSelect() {
    const vendorSelect = document.getElementById("vendorSelect");
    vendorSelect.innerHTML = "<option value=''>--Select Business--</option>";
    vendorData.forEach(vendor => {
      const option = document.createElement("option");
      option.value = vendor.vendorId;
      option.textContent = vendor.vendorId;
      vendorSelect.appendChild(option);
    });
  }

  function saveToFile() {
    const csvContent = vendorData
      .map(vendor => `${vendor.vendorId},${vendor.secretKey},${vendor.encryptedValue}`)
      .join("\n");
    const blob = new Blob([csvContent], { type: "text/csv" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "details.csv";
    a.click();
    URL.revokeObjectURL(url);
  }

  function showDecryptedValue() {
    const vendorId = document.getElementById("vendorSelect").value;
    const secretKey = document.getElementById("decryptSecretKey").value;

    if (!vendorId || !secretKey) {
      alert("Please select a business and enter the secret key.");
      return;
    }

    const vendor = vendorData.find(v => v.vendorId === vendorId);
    if (!vendor) {
      alert("Business not found!");
      return;
    }

    try {
      const bytes = CryptoJS.AES.decrypt(vendor.encryptedValue, secretKey);
      const decryptedValue = bytes.toString(CryptoJS.enc.Utf8);
        if(decryptedValue){
                  document.getElementById("decryptedValue").value = decryptedValue ? decryptedValue : "Incorrect Secret Key";
        }else{
              alert("Please select a business and enter the secret key.");
}
    } catch (e) {
      document.getElementById("decryptedValue").value = "Decryption failed!";
    }
  }

  function clearDecryptedValue() {
    document.getElementById("decryptedValue").textContent = "N/A";
    document.getElementById("decryptSecretKey").value = "";
  }
    
    function copyToClipboard() {
    const decryptedValue = document.getElementById("decryptedValue").textContent;
    navigator.clipboard.writeText(decryptedValue).then(() => {
      alert("Copied to clipboard!");
    }, () => {
      alert("Failed to copy.");
    });
  }
</script>

</body>
</html>
