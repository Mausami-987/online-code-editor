# online-code-editor

//*Save the HTML code as online code editor.html.
Install Express and body-parser in your Node.js environment (npm install express body-parser).
Save the JavaScript code as app.js.
Run the Node.js server by executing node app.js.
Open online code editor.html in your browser.//

//Html code//
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Online Java Code Editor</title>
<style>
    body {
        font-family: Arial, sans-serif;
        margin: 0;
        padding: 0;
    }
    #editor {
        width: 100%;
        height: 300px;
        border: 1px solid #ccc;
        margin-bottom: 10px;
    }
    #output {
        width: 100%;
        min-height: 200px;
        border: 1px solid #ccc;
        padding: 10px;
        box-sizing: border-box;
    }
    button {
        padding: 10px;
        background-color: #007bff;
        color: #fff;
        border: none;
        cursor: pointer;
    }
</style>
</head>
<body>
<div id="editor" contenteditable="true">
    // Write your Java code here...
</div>
<button onclick="runCode()">Run</button>
<div id="output"></div>
<script>
    function runCode() {
        var code = document.getElementById('editor').innerText;
        var xhr = new XMLHttpRequest();
        xhr.open('POST', '/compile', true);
        xhr.setRequestHeader('Content-Type', 'application/json');
        xhr.onreadystatechange = function () {
            if (xhr.readyState === XMLHttpRequest.DONE) {
                if (xhr.status === 200) {
                    document.getElementById('output').innerText = xhr.responseText;
                } else {
                    document.getElementById('output').innerText = 'Error: ' + xhr.responseText;
                }
            }
        };
        xhr.send(JSON.stringify({ code: code }));
    }
</script>
</body>
</html>



//JavaScript code//
const express = require('express');
const { spawn } = require('child_process');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json());

app.post('/compile', (req, res) => {
    const code = req.body.code;
    const javac = spawn('javac', ['-'], { shell: true });
    const java = spawn('java', ['-cp', '.', 'Main'], { shell: true });

    javac.stdin.write(code);
    javac.stdin.end();

    var compilationError = '';
    javac.stderr.on('data', data => {
        compilationError += data.toString();
    });

    javac.on('close', () => {
        if (compilationError) {
            res.status(200).send(compilationError);
        } else {
            java.stdin.write(code);
            java.stdin.end();
            var output = '';
            java.stdout.on('data', data => {
                output += data.toString();
            });
            java.stderr.on('data', data => {
                output += data.toString();
            });
            java.on('close', () => {
                res.status(200).send(output);
            });
        }
    });
});

app.listen(3000, () => console.log('Server running on port 3000'));


