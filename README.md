const fileInput = document.getElementById('fileInput');
const generateBtn = document.getElementById('generateBtn');
const downloadBtn = document.getElementById('downloadBtn');
const previewImage = document.getElementById('previewImage');
const qrCanvas = document.getElementById('qrCanvas');
const recordText = document.getElementById('recordText');
const recordTitle = document.getElementById('recordTitle');
const qrPanel = document.getElementById('qrPanel');

let currentImageData = '';
let currentFileName = 'attestation';

fileInput.addEventListener('change', (event) => {
  const file = event.target.files?.[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = (e) => {
    currentImageData = String(e.target?.result || '');
    currentFileName = file.name.replace(/\.[^/.]+$/, '') || 'attestation';
    previewImage.src = currentImageData;
    previewImage.alt = file.name;

    if (currentImageData) {
      recordTitle.textContent = currentFileName;
    }
  };

  reader.readAsDataURL(file);
});

generateBtn.addEventListener('click', async () => {
  const payload = buildQrPayload();

  if (!payload) {
    alert('Please upload an attestation image first.');
    return;
  }

  qrPanel.classList.remove('hidden');
  recordTitle.textContent = currentFileName;

  try {
    await QRCode.toCanvas(qrCanvas, payload, {
      width: 220,
      margin: 2,
      color: {
        dark: '#111827',
        light: '#ffffff'
      }
    });
  } catch (error) {
    console.error('QR generation failed:', error);
    alert('Unable to generate QR. Please try again.');
  }
});

downloadBtn.addEventListener('click', () => {
  const link = document.createElement('a');
  link.href = qrCanvas.toDataURL('image/png');
  link.download = `${currentFileName || 'attestation'}-qr.png`;
  link.click();
});

function buildQrPayload() {
  const customText = recordText.value.trim();
  const payload = {
    name: currentFileName || 'attestation',
    type: 'attestation',
    source: currentImageData ? 'uploaded-image' : 'manual-entry',
    notes: customText || 'No extra record details supplied',
    generatedAt: new Date().toISOString(),
    imageData: currentImageData || null
  };

  if (!currentImageData) {
    return null;
  }

  return JSON.stringify(payload, null, 2);
}
