# DataGrid
DataGrid for Nette Framework based on Niftyx/NiftyGrid with Twitter Bootstrap support

This is extended version of NiftyGrid - a simple and light-weight **datagrid** for *Nette framework*.
This is optimalized for Nette v2.2 and v2.1. Support for Twitter Bootstrap 2 and 3.


## Install
Extract zip file to vendor/others/NiftyGrid
Copy resources from `resources/` directory to your public www dir and include them in template (usually in `@layout.latte`).
For AJAX use [nette.ajax.js](http://addons.nette.org/vojtech-dobes/nette-ajax-js) and add selector .grid-ajax :

```
$.nette.init(function (ajaxHandler) {
	$('.grid-ajax').on('click', ajaxHandler);
});
```

## Usage

See manual on http://addons.nette.org/cs/niftygrid
