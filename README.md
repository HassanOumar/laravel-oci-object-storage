## Custom Laravel FileSystem for Oracle Object Storage

Clone this repository or create your fresh Laravel project

Add the following keys in your **.env** file 

    FILESYSTEM_DRIVER=oci  
    OCI_ACCESS_KEY_ID=  
    OCI_SECRET_ACCESS_KEY=  
    OCI_DEFAULT_REGION=  
    OCI_BUCKET=  
    OCI_URL=

Get the values from your Oracle console.

Note the the URL format should be as follow

    https://{{Namespace}}.compat.objectstorage.{{region}}.oraclecloud.com
Get the Namespace from your bucket details or from your user page from the console

Then update **config/filesystems.php**, add the following values

	'disks' => [
		...
	    'oci' => [
            'driver' => 's3',
            'key' => env('OCI_ACCESS_KEY_ID'),
            'secret' => env('OCI_SECRET_ACCESS_KEY'),
            'region' => env('OCI_DEFAULT_REGION'),
            'bucket' => env('OCI_BUCKET'),
            'url' => env('OCI_URL') .  '/'.  env('OCI_BUCKET'),
	    ],
	]

Then run the following command your project to create the provider

    php artisan make:provider OciObjectStorageServiceProvider

Then update the created file as follow

    use Aws\S3\S3Client;
    use League\Flysystem\AwsS3v3\AwsS3Adapter;
    use League\Flysystem\Filesystem;
    use Storage;
    ...
    public  function  boot()
    {
	    if (config('filesystems.default') != 'oci') {
		    return;
	    }
    
	    Storage::extend('s3', function($app, $config) {
		    $client = new  S3Client([
			    'credentials' => [
				    'key' => $config['key'],
				    'secret' => $config['secret'],
			    ],
			    'region' => $config['region'],
			    'version' => '2006-03-01',
			    'bucket_endpoint' => true,
			    'endpoint' => $config['url']
		    ]);
    
		    return  new  Filesystem(new  AwsS3Adapter($client, $config['bucket'], $config['bucket']));
	    });
    }

Then finally just add the newly created provider in **config/app.php**

    'providers' => [
        ...
        App\Providers\OciObjectStorageServiceProvider::class,
        ...
	],

I hope it was clear.

Happy Coding...
