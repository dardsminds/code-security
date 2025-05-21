PHP
```
<?php
  $userInput = $_GET['input'];
  echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
?>
```

JavaScript
```
// Safe
document.getElementById('output').textContent = userInput;

// Unsafe
document.getElementById('output').innerHTML = userInput;
```

NodeJS
```
const { body } = require('express-validator');

app.post('/comment', 
  body('text').escape(), // Sanitizes input
  (req, res) => {
    // Safe to use req.body.text now
  }
);
```
