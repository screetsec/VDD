# Stored Cross-site scripting (XSS)

It was discovered that the application does not validate user input and lacks implementation of sanitization for several parameters, leaving it susceptible to Cross-Site Scripting (XSS) attacks.

### **Affected Parameters**

General Data

- sitename

Default Template Setting

- ogImage
- itemsHeader
- brand
- placeholderSearch
- iconNav
- itemsFooter
- SearchResults

Default Colors:

- colorPageText
- colorPageBackground
- colorPageBorder
- colorCardText
- colorCardBackground
- colorCardBorder
- colorCodeBackground
- colorNavbarText
- colorNavbarBackground
- colorNavbarBorder

### Code Analysis

The vulnerability is present in the **`SharedController`** class, which is responsible for handling form data and saving shared information.

File: src\UI\Controllers\SharedController.php

```jsx
class SharedController {
	/**
	 * Send form when there is no posted data in the request or save data if there is.
	 *
	 * @return Response the response object
	 */
	public static function data() {
		$Automad = UICache::get();

		if ($data = Request::post('data')) {
			// Save changes.
			$Response = self::save($Automad, $data);
		} else {
			// If there is no data, just get the form ready.
			$SharedData = new SharedData($Automad);
			$Response = new Response();
			$Response->setHtml($SharedData->render());
		}

		return $Response;
	}

	/**
	 * Save shared data.
	 *
	 * @param Automad $Automad
	 * @param array $data
	 * @return Response the response object
	 */
	private static function save(Automad $Automad, array $data) {
		$Response = new Response();

		if (is_writable(AM_FILE_SHARED_DATA)) {
			FileSystem::writeData($data, AM_FILE_SHARED_DATA);
			Cache::clear();

			if (!empty($data[AM_KEY_THEME]) && $data[AM_KEY_THEME] != $Automad->Shared->get(AM_KEY_THEME)) {
				$Response->setReload(true);
			} else {
				$Response->setSuccess(Text::get('success_saved'));
			}
		} else {
			$Response->setError(Text::get('error_permission') . '<br /><small>' . AM_FILE_SHARED_DATA . '</small>');
		}

		return $Response;
	}
}
```

In the **`save`** method, the application utilizes **`FileSystem::writeData($data, AM_FILE_SHARED_DATA)`** to write and save the provided data into a file named **`data.txt`** located within the `shared` folder.  

![Untitled](Stored%20Cross-site%20scripting%20(XSS)%20c79ada828ac04f958036896c305dbb9f/Untitled.png)

Below is the relevant information from the **`Config.php`** file, where the file paths and extensions are defined:

**File: automad\src\Core\Config.php (Line: 103, 120)**

```jsx
// Define all constants which are not defined yet by the config file.
		// DIR
		self::set('AM_DIR_PAGES', '/pages');
		self::set('AM_DIR_SHARED', '/shared');
		self::set('AM_DIR_PACKAGES', '/packages');
		self::set('AM_DIR_CACHE', '/cache');
		self::set('AM_DIR_CACHE_PAGES', AM_DIR_CACHE . '/pages');
		self::set('AM_DIR_CACHE_IMAGES', AM_DIR_CACHE . '/images');
		self::set('AM_DIR_TRASH', AM_DIR_CACHE . '/trash');
		self::set('AM_DIRNAME_MAX_LEN', 60); // Max dirname length when creating/moving pages with the UI.

		// FILE
		self::set('AM_FILE_EXT_DATA', 'txt'); // Changing that constant will also require updating the .htaccess file! (for blocking direct access)
		self::set('AM_FILE_EXT_CAPTION', 'caption');
		self::set('AM_FILE_PREFIX_CACHE', 'cached'); // Changing that constant will also require updating the .htaccess file! (for blocking direct access)
		self::set('AM_FILE_EXT_PAGE_CACHE', 'html');
		self::set('AM_FILE_EXT_HEADLESS_CACHE', 'json');
		self::set('AM_FILE_SHARED_DATA', AM_BASE_DIR . AM_DIR_SHARED . '/data.' . AM_FILE_EXT_DATA);
```

The provided code snippet indicates that the variables of the parameters are not sanitized on the client side when rendering the data.

File: packages\standard\templates\post.php

```php
<!DOCTYPE html>
<html lang="en" class="@{ theme | sanitize }">
<head>
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1">

	<!-- Variable not sanitized: sitename -->
	<title>@{ metaTitle | def('@{ sitename } / @{ title | def ("404") }') }</title>
	<@ elements/metatags.php @>
	<@ elements/favicons.php @>
	<# 
	
	To make sure the following variables are always available in the dashboard, 
	they can be included in a comment block.
	
	@{ imageCard } 
	@{ checkboxHideThumbnails }
	
	#>
	<link href="/packages/standard/dist/standard.min.css" rel="stylesheet">
	<script src="/packages/standard/dist/standard.min.js"></script>
	<@ elements/colors_header.php @>
	<# Add optional header items. #>

	<!-- Variable not sanitized: itemsHeader -->
	@{ itemsHeader }
</head>

<body class="@{ :template | sanitize }">
	<@ elements/navbar.php @>
	@{ +hero | replace ('/^(.+)$/is', '<section class="hero content">$1</section>') }
	<div class="uk-container uk-container-center">
		<# 

		Define the default main snippet to display the actual content. 
		The snippet can be overriden before including the actual template in order to extend a template.

		#>
		<@ snippet main @>
			<@ elements/header.php @>
				<main class="content uk-block">
					<@ elements/content.php @>
					<@ elements/prev_next.php @>
				</main>
				<div class="content uk-block">
					<@ elements/related_posts.php @>
				</div>
			<@ elements/footer.php @>
		<@ end @>
		<# 

		Get the output of the main snippet. 

		#>
		<@ main @>
		<footer class="uk-block">
			<div class="am-block footer uk-margin-bottom">
				<ul class="uk-grid uk-grid-width-medium-1-2" data-uk-grid-margin>
					<li>
						<# @{ checkboxShowInFooter } #>
						<@~ newPagelist { 
							excludeHidden: false,
							match: '{ "checkboxShowInFooter": "/[^0]+/" }' 
						} @>
						<@~ foreach in pagelist @>
							<a href="@{ url }"><@ elements/icon_title.php @></a><br />
						<@~ end @>
					</li>
					<li class="uk-text-right uk-text-left-small">
						<a href="/">
							<!-- Variable not sanitized: sitename -->
							&copy; @{ :now | dateFormat('Y') } @{ sitename }
						</a>
					</li>
				</ul>
				<# 

				Add optional footer items. 

				#>
				<!-- Variable not sanitized: itemsFooter  -->
				@{ itemsFooter }
			</div>
		</footer>
	</div>
</body>
</html>
```

Affected Files: 

- packages\standard\templates\post.php (Line:  6, 22, 67, 76)
- packages\standard\templates\elements\navbar.php (Line: 19, 35, 65,
- packages\standard\templates\elements\icon_title.php (Line: 1)
- packages\standard\templates\elements\colors.php (Line: 1-10)
- packages\standard\templates\elements\colors_header.php  (Line: 1-3)

### **Step To Reproduce**

1. Login to the application and navigate to the “General Data and Files” menu 
2. Input the payload on the affected fields or parameter such as `<svg onload=alert("Sitename")//`
    
    ![Untitled](Stored%20Cross-site%20scripting%20(XSS)%20c79ada828ac04f958036896c305dbb9f/Untitled%201.png)
    
3. Default Colors Section:
    
    ![Untitled](Stored%20Cross-site%20scripting%20(XSS)%20c79ada828ac04f958036896c305dbb9f/Untitled%202.png)
    
4. Save the changes. The following screenshots show the request and response from Burp Suite
5. Observe that the homepage will be affected as a result.
    
    ![Untitled](Stored%20Cross-site%20scripting%20(XSS)%20c79ada828ac04f958036896c305dbb9f/Untitled%203.png)
    
    ![Untitled](Stored%20Cross-site%20scripting%20(XSS)%20c79ada828ac04f958036896c305dbb9f/Untitled%204.png)
    
    ![Untitled](Stored%20Cross-site%20scripting%20(XSS)%20c79ada828ac04f958036896c305dbb9f/Untitled%205.png)
