# Unrestricted File Upload

By default, in the **`config.php`** files, the application allows upload files containing d**angerous** types, such as **SVG** and **PDF**.

```jsx
<?php return <<< JSON
{
    "AM_ALLOWED_FILE_TYPES": "dmg, iso, rar, tar, zip, aiff, m4a, mp3, ogg, wav, ai, dxf, eps, gif, ico, jpg, jpeg, png, psd, svg, tga, tiff, avi, flv, mov, mp4, mpeg, css, js, md, pdf",
    "AM_CACHE_ENABLED": true,
    "AM_CACHE_LIFETIME": 43200,
    "AM_CACHE_MONITOR_DELAY": 120,
    "AM_DEBUG_ENABLED": false,
    "AM_FEED_ENABLED": true,
    "AM_FEED_FIELDS": "+hero, +main",
    "AM_FILE_GUI_TRANSLATION": "",
    "AM_HEADLESS_ENABLED": false
}
JSON;
```

The application also not validate the content type, as shown in the code snippets below are associated with the **`upload`** method in the **`FileCollectionController.php`** file, located at **`src\UI\Controllers`**

```jsx
/** src\UI\Controllers\FileCollectionController.php **/

public static function upload() {
		$Automad = UICache::get();
		Debug::log($_POST + $_FILES, 'files');

		// Set path.
		// If an URL is also posted, use that URL's page path. Without any URL, the /shared path is used.
		$path = FileSystem::getPathByPostUrl($Automad);

		return FileCollectionModel::upload($_FILES, $path);
	}
```

Model:  `FileCollectionModel.php`

```jsx
public static function upload(array $files, string $path) {
		$Response = new Response();

		// Move uploaded files
		if (isset($files['files']['name'])) {
			// Check if upload destination is writable.
			if (is_writable($path)) {
				$errors = array();

				// In case the $files array consists of multiple files (IE uploads!).
				for ($i = 0; $i < count($files['files']['name']); $i++) {
					// Check if file has a valid filename (allowed file type).
					if (FileSystem::isAllowedFileType($files['files']['name'][$i])) {
						$newFile = $path . Str::slug($files['files']['name'][$i]);
						move_uploaded_file($files['files']['tmp_name'][$i], $newFile);
					} else {
						$errors[] = Text::get('error_file_format') . ' "' .
									FileSystem::getExtension($files['files']['name'][$i]) . '"';
					}
				}

				Cache::clear();

				if ($errors) {
					$Response->setError(implode('<br />', $errors));
				}
			} else {
				$Response->setError(Text::get('error_permission') . ' "' . basename($path) . '"');
			}
		}

		return $Response;
	}
```

This issue allow pentester to upload a SVG or PDF file contains malicious content to **execute** arbitrary JS code which acts as a stored XSS payload.

SVF File:

![1](https://github.com/screetsec/VDD/assets/17976841/39bed2a9-3dcb-4268-9d61-aff36e49bb57)

![2](https://github.com/screetsec/VDD/assets/17976841/b5deb569-0303-4c5b-bb93-f184beb509c1)

PDF File:

![3](https://github.com/screetsec/VDD/assets/17976841/68fbd4fc-dead-42e0-8a76-a3e9320c4202)

![4](https://github.com/screetsec/VDD/assets/17976841/0196148c-fc9f-486f-abdb-87b80d6ac8bc)
