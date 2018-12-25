# html to image

### canvas to image

#### install
`npm install --save canvas2image`

#### 使用
```javascript
import html2canvas from 'html2canvas';
html2canvas(document.querySelector('#node')).then((canvas) => {
  const link = document.createElement('a');
  link.download = 'filename.png';
  link.href = canvas.toDataURL('image/jpeg', 1);
  link.click();
});
```

#### 问题
* dash border 展示成 solid。

### dom to image

#### install
`npm install --save dom-to-image`

#### 使用
```javascript
import domtoimage from 'dom-to-image';
const node = document.querySelector('#report-content');
domtoimage.toPng(node)
  .then((dataUrl) => {
    const link = document.createElement('a');
    link.download = 'filename.png';
    link.href = dataUrl;
    link.click();
  })
  .catch((error) => {
    console.error('dom to image error!', error);
  });
```