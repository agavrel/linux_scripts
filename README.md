# Linux Scripts Collection by agavrel

> Obviously you will need to install the corresponding libraries to make this scripts work like a charm.

---
### Get remaining size in the cpu

    df -h
    
---
### Get largest files in the directory

    ls -lhSr

---
### Get largest directories in the directory

    du -d | sort -n
    
---
### Reduce brightness below keyboard possible level

    echo 50 | sudo tee /sys/class/backlight/intel_backlight/brightness

---
### Create favicon from image (give image as argument to the script)
    #!/bin/sh
    if [ ! -z $2 ]
    then
        echo "creating file ${2}.ico"
        OUTPUT_FILE="${2}.ico"
    else
        echo 'creating file favicon.ico'
        OUTPUT_FILE='favicon.ico'
    fi

    # ./create_favicon.sh portal_logo.png
    /usr/bin/convert $1 \
                    -resize x16 -gravity center -crop 16x16+0+0 \
                    $OUTPUT_FILE

    # -transparent white



---

### Combine .pdf files

```
qpdf --empty --pages *.pdf -- out.pdf
```

### Convert .md to .pdf

#### Convert .md to pdf

    sudo apt-get install pandoc texlive-latex-base texlive-fonts-recommended texlive-extra-utils
    sudo apt-get install texlive-xetex
    fc-list :lang=zh
    pandoc --from markdown-yaml_metadata_block README.md -o JapanTrip.pdf --latex-engine=xelatex -V geometry:margin=1in -V CJKmainfont="Noto Serif CJK JP"

PS: add '\pagebreak' to force pagebreak in the resulting .pdf  
PS2: To include images use following syntax: ![Map of Kyoto](img/kyoto.png)\

#### Get Fullsize Images from Instagram

https://www.instagram.com/

Right click on image then view page source then look the first occurence of .jpg and copy the full url which should look like https://scontent-ort2-1.cdninstagram.com/vp/b3c78497f8058952dda19afa4a9c8c0d/5E1F1B14/t51.2885-15/e35/s1080x1080/70811296_2836832639677828_5645193438355127253_n.jpg?_nc_ht=scontent-ort2-1.cdninstagram.com&_nc_cat=104  

#### Image Resizing
    convert nara_national_museum.jpg -resize 200% nara_national_museum2.jpg
    identify -format "%w %h" nara_national_museum2.jpg
    convert nara_national_museum2.jpg -trim  -gravity North -crop 2560x1200+0+0 +repage nara_national_museum3.jpg



---
### Imagemagick

#### Get image format
    identify -format "%w x %h %x x %y" 1.png

#### Adjust DPI (pixel per inch) to 300
    convert -units PixelsPerInch 1.png -resample 300 2.png

#### Crop Image to 600x600 with 70 pixels offset
    convert -crop 600x600+70 2.png 3.png

#### Fill the A4 page with the picture
    convert 3.png 3.png 3.png +append 4.png
    convert 4.png 4.png -append 5.png

#### adjust DPI as PixelsPerCentimeter instead (3.5x4.5cm)
    convert test1.jpg -crop 1652x2124+310 test2.jpg
    magick test2.jpg -units PixelsPerCentimeter -density %[fx:w/3.5] test3.jpg

#### Make image transparent
    convert $1 -fuzz 10% -transparent white result.png


---
### Ultrafast download/upload
    parallel rsync -aviP --compress-level=7 $1 :::  ubuntu@52.81.47.113:folder_name/

---
### Open current folder location  
    gio open .

---
### Download youtube music playlist

You first need a python file that will get all the youtube IDs contained in the playlist of your choice:

    from selenium import webdriver
    import time
    from pyvirtualdisplay import Display
    import logging
    import traceback
    import re
    from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC
    from selenium.webdriver.common.action_chains import ActionChains
    from selenium.webdriver.common.by import By
    import pickle
    import sys

    options = webdriver.ChromeOptions()
    options.add_argument('--start-maximized')
    options.add_argument('--auto-open-devtools-for-tabs')
    caps = DesiredCapabilities().CHROME
    #caps["pageLoadStrategy"] = "none"
    driver = webdriver.Chrome(
        desired_capabilities=caps,
        executable_path='/usr/lib/chromium-browser/chromedriver',
        service_args=['--verbose', '--log-path=./chromelog.log'],
        options=options
    )

    playlist = "https://www.youtube.com/playlist?list=PLef6KwWToCfW_Vwdpy3eHa1Z8_M5uw9sW"
    if len(sys.argv) > 1:
        playlist = sys.argv[1]
    driver.get(playlist)
    myclass = "yt-simple-endpoint"
    subclass = "style-scope"
    subclass2 = "ytd-playlist-video-renderer"
    mylist = driver.find_elements_by_css_selector(f".{myclass}.{subclass}.{subclass2} a")#.get_attribute('href')
    newlist = ""
    i = 0
    for a in mylist:
        b = a.get_attribute('href')
        start = "watch?v="
        end = "&list"
        pattern = rf"({start}).*?({end})"
        pattern = f"watch\?v=(.*?)&list"
        c = re.search(pattern, b)
        if c:
            i += 1  #print(c.group(1))
            newlist += c.group(1) + '\n'
    myfile = "playlist.txt"
    with open(myfile, 'w') as f:
        f.write(newlist)

    print(f"\033[92m\033[5mSucessfully got {i} playlist track IDs from {playlist}!\033[0m")

    dalist = [line.rstrip('\n') for line in open(myfile, 'r')]
    print(dalist)


which will generate a file containing the playlist IDs:

    cat ~/Music/playlist.txt
    nEAlV-obkFY
    Xh4ugYiXF-Q
    mRKTOZmX2cE

##### download_youtube_playlist.sh

    #!/bin/sh
    for word in $(cat ~/Music/playlist.txt);
    #do youtube-dl --extract-audio --audio-format mp3
    do youtube-dl --format bestaudio[ext=m4a] https://www.youtube.com/watch?v=$word;
    done

##### download_youtube_mp3.sh

    #!/bin/sh
    youtube-dl --extract-audio --audio-format mp3 $1

##### download_youtube_m4a.sh

    #!/bin/sh
    youtube-dl --format bestaudio[ext=m4a] $1


---
### Connect to remote AWS server through ssh port

Make sure that port 22 is opened in AWS security group, also change the IP to the elastic IP that was set:

    ssh -v -i "aws_key.pem" ubuntu@15.236.44.243

### Send a file to AWS server using SCP

$1 is folder location and $2 filename, also change the IP to the elastic IP that was set:

    sudo scp -i "aws_key.pem" $1/$2 ubuntu@15.236.44.243:$2


---
### Create Layer in AWS using CLI

#### Create a [virtualenv](https://uoa-eresearch.github.io/eresearch-cookbook/recipe/2014/11/26/python-virtual-env/)

You have to first package libraries with the right python environment or you will end up with a package in Python 2.7 which will not work with your lambda:

    PYVERS=3.7 && \
    sudo apt-get install python$PYVERS-dev && \
    sudo apt-get install python$PYVERS-venv \
    && sudo virtualenv -p /usr/bin/python$PYVERS work$PYVERS
    && source work$PYVERS/bin/activate

#### Packaging the library

Now package the library into a layer.zip archive:

    pip3 install --upgrade -t ./python python-dateutil==2.8.0 boto3 requests regex xlrd urllib3 pandas


If you want a specific version you have to do this way:
    pip3 install --upgrade -t ./python  regex==2019.08.19


Finally send it to the s3 bucket, replace 'mybucket' by your bucket's name

    zip -r9 layer.zip ./python && \
    aws s3 cp layer.zip s3://mybucket/layers/

_NB: it is __very important to keep the name python__ as the folder name for the package has to match to language, for the zip file it does not matter_

#### Publishing the library

Then publish the the layer containing the desired library with:

    LAYER=mylayer && BUCKET_NAME=mybucket \
    && aws lambda publish-layer-version \
        --layer-name $LAYER \
        --description "library package" \
        --license-info "MIT" \
        --content S3Bucket=$BUCKET_NAME,S3Key=layers/layer.zip \
        --compatible-runtimes python$PYVERS'


---
### React.js
#### Create blank app
    npx create-react-app myapp

#### Starts the development server
    npm start


#### Bundles the app into static files for production
    npm run build


#### Starts the test runner
    npm test


#### Removes this tool and copies build dependencies, configuration files and scripts into the app directory. If you do this, you can’t go back!
    npm run eject
