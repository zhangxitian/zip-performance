# node/js zip性能对比(1071个512k的文件)

``` javascript
  // 1 level6/87s level3/42s
  let zip = new jszip()
  readDir(zip, 'D:\\tmp')
  zip.generateAsync({
    type: 'nodebuffer',
    compression: "DEFLATE",
    compressionOptions: {
      level: 3
    }
  }).then(content => {
    // 可直接上传
    fs.writeFile('D:/tmp2/result.zip', content, err => {
      console.log('完成' + (+new Date()))
    })
  }).catch(err => {
    console.log('打包错误')
    return 'error'
  })
```
``` javascript
  // 2 level3/42s level6/88.7s
  let zip = new jszip()
  readDir(zip, 'D:\\tmp')
  zip
  .generateNodeStream({
    type: 'nodebuffer',
    compression: "DEFLATE",
    compressionOptions: {
      level: 3
    },
    streamFiles:true
  })
  .pipe(fs.createWriteStream('D:/tmp2/result.zip'))
  .on('finish', function () {
     // JSZip generates a readable stream with a "end" event,
     // but is piped here in a writable stream which emits a "finish" event.
     console.log('完成' + (+new Date()))
  })
```
``` javascript

  //3 level6/2.55m level3/2.17m
  fstream.Reader({ 'path': 'D:/tmp', 'type': 'Directory' }) /* Read the source directory */
  .pipe(tar.Pack()) /* Convert the directory to a .tar file */
  .pipe(zlib.Gzip({
     level: 3
  })) /* Compress the .tar file */
  .pipe(fstream.Writer({path: 'D:/tmp2/result.zip'}))
```

``` javascript
  // 4 level6/63s level3/29s
  const archive = archiver('zip', {
    zlib: { level: 3 } // Sets the compression level.
  })
  archive.directory('D:/tmp', false)
  const output = fs.createWriteStream('D:/tmp2/result.zip')
  archive.pipe(output).on('finish', function() {
    console.log('完成' + (+new Date()))
  })
  output.on('end', function() {
    console.log('Data has been drained');
  })
  archive.finalize()
```
