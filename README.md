nginx-upload-module
===================

This module is based on Nginx upload module (v 2.2.0) http://www.grid.net.ru/nginx/upload.en.html. ...
Since it seems the author has not maintained that module. I changed some codes that can be installed with latest nginx.

- install
./configure --add-module={module_dir} && make && make install

- conf
```
server {
    listen       80;
    client_max_body_size 100m;

    location / {
        root  html/upload;
    }

    # Upload form should be submitted to this location
    location /upload {
        # Pass altered request body to this location
        upload_pass   /example.php;

        # Store files to this directory
        # The directory is hashed, subdirectories 0 1 2 3 4 5 6 7 8 9 should exist
        upload_store /tmp/upload 1;

        # Allow uploaded files to be read only by user
        upload_store_access user:r;

        # Set specified fields in request body
        upload_set_form_field "${upload_field_name}_name" $upload_file_name;
        upload_set_form_field "${upload_field_name}_content_type" $upload_content_type;
        upload_set_form_field "${upload_field_name}_path" $upload_tmp_path;

        # Inform backend about hash and size of a file
        upload_aggregate_form_field "${upload_field_name}_md5" $upload_file_md5;
        upload_aggregate_form_field "${upload_field_name}_size" $upload_file_size;

        upload_pass_form_field "^submit$|^description$";
    }

    location ~ \.php$ {
        root           html/upload;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```
