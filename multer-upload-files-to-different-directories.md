# Use [Multer](https://github.com/expressjs/multer) to Upload Files to Different Directories

Is this possible? Yes, with Express.

Use the official [multer](https://github.com/expressjs/multer) distro via npm. The example below uses Express 4.

### Key points in source code

The key is to add multer as specific middleware to each route you want parsed.

```js
var multer  = require('multer');

var mwMulter1 = multer({ dest: './uploads1/' });

app.post('/files1', mwMulter1, function(req, res) {
	// check req.files for your files
});

var mwMulter2 = multer({ dest: './uploads2/' });

app.post('/files2', mwMulter2, function(req, res) {
	// check req.files for your files
});
```

### Security Benefit

As [james147 points out](https://github.com/expressjs/multer/issues/59), as of Multer version 0.1.4, putting an instance of Multer in the global scope of all processing creates a security risk. Multer will run for any post/put request, even when the developer intends for the targeted page to not receive files. This can be exploited by a hacker to create a Denial of Service attack.

By placing the Multer instance on the exact route, the developer can narrow the scope of the attack to the route. 

### Using the Source Code

Make a test directory and place the full source below in a server.js file. 

Then npm install the following libaries:

```bash
$ npm install express
$ npm install body
$ npm install express
```

Run the server

`$ node server2`

Open two browser windows (or tabs) with the following urls:

1. [http://localhost/8080/files1](http://localhost/8080/files1): files are uploaded to the uploads1/ directory
2. [http://localhost/8080/files2](http://localhost/8080/files2): files are uploaded to the uploads2/ directory
3. [http://localhost/8080/filesFail](http://localhost/8080/filesFail): the upload cannot be parsed. This is good. No security leak.

### Source Code (Full Example)

```js
var express    = require('express'); 		// call express
var multer  = require('multer');
var util = require('util');

var port = process.env.PORT || 8080; 		// set our port

var app = express();

app.get('/files1', function(req, res) {
		console.log('IN GET');
		res.writeHead(200, { Connection: 'close' });
		res.end('<html><head></head><body>\
		               <form method="POST" enctype="multipart/form-data">\
		                <input type="text" name="textfield1"><br />\
		                <input type="file" multiple name="file1"><br />\
		                <input type="submit">\
		              </form>\
		            </body></html>');
	});

// middleware function placed directly on route, initialized by multer
var mwMulter1 = multer({ dest: './uploads1/' });

app.post('/files1', mwMulter1, function(req, res) {

		console.log('IN POST (/files1)');
		console.log(req.body)

		var filesUploaded = 0;

		if ( Object.keys(req.files).length === 0 ) {
			console.log('no files uploaded');
		} else {
			console.log(req.files)

			var files = req.files.file1;
			if (!util.isArray(req.files.file1)) {
				files = [ req.files.file1 ];
			} 

			filesUploaded = files.length;
		}

		res.json({ message: 'Finished! Uploaded ' + filesUploaded + ' files.  Route is /files1' });
	});

app.get('/files2', function(req, res) {
		console.log('IN GET');
		res.writeHead(200, { Connection: 'close' });
		res.end('<html><head></head><body>\
		               <form method="POST" enctype="multipart/form-data">\
		                <input type="text" name="textfield1"><br />\
		                <input type="file" multiple name="file1"><br />\
		                <input type="submit">\
		              </form>\
		            </body></html>');
	});

// middleware function placed directly on route, initialized by multer
var mwMulter2 = multer({ dest: './uploads2/' });

app.post('/files2', mwMulter2, function(req, res) {

		console.log('IN POST (/files2)');
		console.log(req.body)

		var filesUploaded = 0;

		if ( Object.keys(req.files).length === 0 ) {
			console.log('no files uploaded');
		} else {
			console.log(req.files)

			var files = req.files.file1;
			if (!util.isArray(req.files.file1)) {
				files = [ req.files.file1 ];
			} 

			filesUploaded = files.length;
		}

		res.json({ message: 'Finished! Uploaded ' + filesUploaded + ' files.  Route is /files2' });
	});

app.get('/filesFail', function(req, res) {
		console.log('IN GET /filesFail');
		res.writeHead(200, { Connection: 'close' });
		res.end('<html><head></head><body>\
		               <form method="POST" enctype="multipart/form-data">\
		                <input type="text" name="textfield1"><br />\
		                <input type="file" multiple name="file1"><br />\
		                <input type="submit">\
		              </form>\
		            </body></html>');
	});

app.post('/filesFail', function(req, res) {

		console.log('IN POST (/filesFail)');
		console.log('req.body = ' + req.body); // this is undefined b/c a body-parser has not been specified
		console.log('req.files = ' + req.files);

		if (req.body === undefined && req.files === undefined )
			res.json({ message: '/filesFail was NOT PARSED... no security leak' });
		else 
			res.json({ message: '/filesFail was parsed.... security leak' });
	});

app.listen(port);
console.log('Started tests ' + port);
```

## License

[The MIT License (MIT)](http://www.opensource.org/licenses/MIT)

Copyright (c) 2014 Jaret Pfluger

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
