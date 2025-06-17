Great, thanks Darlene — here’s the specific example for your stack:

✅ IdP: Azure AD
✅ Webserver: Apache
✅ PHP: 8.2
✅ SimpleSAMLphp: usually installed at /var/simplesamlphp/
✅ You’ll likely want SP-initiated login — user starts from your PHP app.

⸻

Full Example: SimpleSAMLphp + Azure AD + PHP App

⸻

📁 Folder Structure (example):

/var/www/html/your-php-app/      --> Your app
/var/simplesamlphp/              --> SimpleSAMLphp


⸻

1️⃣ Apache Config

Enable URL rewrite for SimpleSAMLphp:

Alias /simplesaml /var/simplesamlphp/www

<Directory /var/simplesamlphp/www>
    Require all granted
</Directory>

Reload Apache:

sudo systemctl reload apache2


⸻

2️⃣ SimpleSAMLphp: config.php

Edit /var/simplesamlphp/config/config.php:

'baseurlpath' => 'simplesaml/',
'certdir' => 'cert/',
'auth.adminpassword' => 'SuperSecretAdminPassword123!',
'secretsalt' => 'RandomGeneratedSaltHere',


⸻

3️⃣ SimpleSAMLphp: authsources.php

Edit /var/simplesamlphp/config/authsources.php:

<?php

$config = [

    'default-sp' => [
        'saml:SP',
        'entityID' => 'https://your-app.local/simplesaml/module.php/saml/sp/metadata.php/default-sp',
        'idp' => 'https://login.microsoftonline.com/{TENANT-ID}/v2.0', // Azure AD
        'privatekey' => 'default-sp.pem',
        'certificate' => 'default-sp.crt',
        'NameIDPolicy' => 'urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress',
    ],

];

👉 Important: replace {TENANT-ID} with your Azure AD Directory (tenant) ID.

⸻

4️⃣ Upload SP Metadata to Azure AD
	1.	Open:
https://your-app.local/simplesaml/module.php/saml/sp/metadata.php/default-sp
	2.	Register the app in Azure AD Enterprise Applications → Single sign-on (SAML).
	3.	Upload your SP metadata there.

⸻

5️⃣ PHP App Example (index.php)

<?php
require_once('/var/simplesamlphp/lib/_autoload.php');

// Create SimpleSAML_Auth_Simple object
$auth = new SimpleSAML\Auth\Simple('default-sp');

// Require user to authenticate
$auth->requireAuth();

// Fetch user attributes
$attributes = $auth->getAttributes();
?>

<!DOCTYPE html>
<html>
<head>
    <title>SSO with SimpleSAMLphp & Azure AD</title>
</head>
<body>
    <h1>Welcome <?php echo htmlspecialchars($attributes['givenName'][0] ?? 'User'); ?>!</h1>

    <h2>Your SAML Attributes:</h2>
    <pre><?php print_r($attributes); ?></pre>

    <p><a href="logout.php">Logout</a></p>
</body>
</html>


⸻

6️⃣ Logout (logout.php)

<?php
require_once('/var/simplesamlphp/lib/_autoload.php');

$auth = new SimpleSAML\Auth\Simple('default-sp');

// Logout, return to homepage
$auth->logout(['ReturnTo' => 'https://your-app.local/index.php']);


⸻

7️⃣ Testing

✅ Open: https://your-app.local/index.php
✅ You will be redirected to Azure AD login
✅ On success, user lands back on your app with SAML attributes

⸻

Final Notes:

✅ Azure AD requires SP metadata registration — that’s the only manual step
✅ SimpleSAMLphp v2.x+ works fine with PHP 8.2
✅ You can customize attribute mapping — I can show how if you want
✅ Azure returns attributes like:

[
    'http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress' => ['user@example.com'],
    'givenName' => ['FirstName'],
    'surname' => ['LastName'],
    'displayName' => ['FirstName LastName'],
    'groups' => ['group1', 'group2'],
]


⸻

Would you like me to also:

1️⃣ Show you how to generate SP certificate (private key + cert) for this?
2️⃣ Show how to map Azure AD claims to friendlier attribute names (so you can write $attributes['email'])?
3️⃣ Give a docker-compose version if you want to run it locally?

Just tell me! 🚀
