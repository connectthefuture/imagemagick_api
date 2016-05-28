# Bitfusion Mobile Image Manipulation Service API

This AMI comes pre-installed with a simple REST API allowing for complex image transformation tasks. Achieve optimal performance by using this AMI on any AWS instance that fits your needs the best. Minimize resource and power consumption on your mobile devices and thin clients by using the RESTful API.


## API



| METHOD       | URI                                                            | ACTION                               |
|--------------|----------------------------------------------------------------|---------------------------------------
| POST         | http://[hostname]/input/                                       | Upload a input file                  | 
| GET          | http://[hostname]/input/                                       | List all input files                 |
| DELETE       | http://[hostname]/input/{file_name}                            | Delete input file                    |
| GET          | http://[hostname]/input/{file_name}                            | Get input file                       |
| GET          | http://[hostname]/job/{tasks}/{image_name}                     | Get job status based on the job id   |
| GET          | http://[hostname]/output/                                      | List all output files                |
| DELETE       | http://[hostname]/output/{file_name}                           | Delete input file                    |
| GET          | http://[hostname]/output/{file_name}                           | Get output file                      |


### API Token

When the system starts, the API token will be set to the instance-id.  You will need to pass the
header 'X-Auth-Token' and have it set to the instance ID for any of the calls to function.

    
### Supported Image Transformations

| task          | option values               | url example:                     |im example command:                                               |
|-------------- |-----------------------------|----------------------------------|------------------------------------------------------------------|
| output_file   | [jpg,png,tiff,gif,pdf]      | /output_f=tiff/image.png         | convert input_file output_file.output_f                          |
| resize_percent| pos integer                 | /resize_percent=110/image.png    | convert input_file -resize integer% output_file                  |
| resize_x      | pos integer                 | /resize_x=20/image.png           | convert input_file -resize integer output_file                   |
| resize_y      | pos integer                 | /resize_y=20/image.png           | convert input_file -resize xinteger output_file                  |
| resize_xy     | pos integer1, pos integer2  | /resize_xy=100,100/image.png     | convert input_file -resize integer1xinteger2 output_file        |
| rotate        | pos or neg float            | /rotate=-270/image.png           | convert input_file -rotate integer1 output_file                  |
| charcoal      | pos integer                 | /charcoal=10/image.png           | convert input_file -charcoal integer1 output_file                |
| negate        |                             | /negate/image.png                | convert input_file -negate                                       |
| grayscale     |                             | /grayscale/image.png             | convert input_file -colorspace gray output_file                  |
| flip_y        |                             | /flip_y/image.png                | convert input_file -flip output_file                             |
| flip_x        |                             | /flip_x/image.png                | convert input_file -flop output_file                             |
| border        | pos integer                 | /border=2/image.png              | convert input_file -border integer output_file                   |
| border_x      | pos integer                 | /border_x=2/image.png            | convert input_file -border integerx0 output_file                 |
| border_y      | pos integer                 | /border_y=2/image.png            | convert input_file -border 0xinteger output_file                 |
| border_xy     | pos integer1, pos integer2  | /border_xy=2/image.png           | convert input_file -border integer1xinteger2 output_file         |
| border_color  | hex-value                   | /bordere_color=#e1e1e1/image.png | convert input_file -bordercolor "#hhhhhh" output_file            |
| blur          | pos float                   | /blur=10/image.png               | convert input_file -blur 0xfloat output_file                     |
| sharpen       | pos float                   | /sharpen=50/image.png            | convert input_file -sharpen 0xfloat output_file                  |
| sepia         | pos integer (1 to 100)      | /sepia=100/image.png             | convert input_file -sepia-tone integer% output_file              |
| crop          | pos int1, int2, int3, int4  | /crop/0,1,50,1/image.png         | convert input_file -crop int1xint2+int3+int4 +repage output_file |

* note 1: bordercolor must come before border argument
* note 2: crop operators are: xoffset, yoffset, xsize, ysize

### Walkthrough
#### Step 1 - Upload a file

```
#!/usr/bin/env bash

token="12345"
host_ip="52.87.180.179"
file="house.jpg"

curl -X POST \
-H "X-Auth-Token: ${token}" \
-H "Cache-Control: no-cache" \
-H "Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW" \
-F "file=@${file}" \
http://${host_ip}:5000/input/

# Expected output:
# {
#     "filename": "house.jpg",
#     "status": "success"
# }

```
#### Step 2 - Transform the file
```
#!/usr/bin/env bash

token="<ec2 instance id>"
host_ip="<ec2 instance public IP>"
file="dubai_skyscraper.jpg"
file_head=$( echo ${file} | cut -d '.' -f1)
file_tail=$( echo ${file} | cut -d '.' -f2)

output_file=${file_head}-out.${file_tail}
echo "Output File: $output_file"

# Image Transform Tasks
task1='border_color=4321e1'
task2='border=20'
task3='rotate=2'

curl -o ${output_file} \
    -X GET \
    -H "X-Auth-Token: ${token}" \
    -H "Cache-Control: no-cache" \
    "http://${host_ip}:5000/job/${task1}/${task2}/${task3}/${file}"
                                                            
    
```  

 * Note 1: By default the file returned will have the same filename.  In the example provided above we specically add "-out" to the filename. 
 * Note 2: Subsequent calls on the same file-name are not cummulative. That is, the transformation always starts with the originally uploaded image. If you want to execute multiple transformations on the same image, make them all part of the same command.
 * Note 3: When multiple transformations are specified, they are executed in the order in which they were specified in the request.

### Postman

We created some sample calls via Postman.  You will need to update an environment and add the url, token, and file variable where:
 * url = {ec2 instance ip:5000}
 * token = {ec2 instance id}
 * file = {name of file you upload}
 
[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/f5d064950c0b48ebb562#?env%5Baws%5D=W3sia2V5IjoidG9rZW4iLCJ2YWx1ZSI6ImktNDg2NDk5ZDQiLCJ0eXBlIjoidGV4dCIsImVuYWJsZWQiOnRydWUsImhvdmVyZWQiOmZhbHNlfSx7ImtleSI6InVybCIsInZhbHVlIjoiNTQuMjM1LjIyNi4yMzI6NTAwMCIsInR5cGUiOiJ0ZXh0IiwiZW5hYmxlZCI6dHJ1ZSwiaG92ZXJlZCI6ZmFsc2V9LHsia2V5IjoidGFza19pZCIsInR5cGUiOiJ0ZXh0IiwidmFsdWUiOiIzNzliYTY1Ny0yYmE5LTRkZWMtOTQ4ZS03YmVhMDhiNWExNGEiLCJlbmFibGVkIjp0cnVlLCJob3ZlcmVkIjpmYWxzZX0seyJrZXkiOiJvdXRwdXRfZmlsZSIsInR5cGUiOiJ0ZXh0IiwidmFsdWUiOiJCTVcyNy1kYWIxYTM5ZC01OWNkLTRkN2QtYjRhZi1jYjEwMDcyZGIyNjctMDEucG5nIiwiZW5hYmxlZCI6dHJ1ZSwiaG92ZXJlZCI6ZmFsc2V9XQ==)

### Swagger
