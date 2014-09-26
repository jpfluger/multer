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

By placing the Multer instance on the exact route, the developer can better prevent this attack.

### Source Code (Full Example)

```js
var express    = require('express'); 		// call express
var app        = express(); 				// define our app using express
var bodyParser = require('body-parser');

var port = process.env.PORT || 8080; 		// set our port

var multer  = require('multer');
var util = require('util');

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

app.listen(port);
console.log('Started tests ' + port);
```