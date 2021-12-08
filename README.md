# Image Comparator and Classifier

Purpose: Set up a static webpage and server to host classifier/pairwise comparator tasks for images 

I am not the original author of these files. This repository contains aggregated and updated files from previous developers. See acknowledgements below.


## Instructions for setup

* [Installation](https://github.com/QTIM-Lab/Image-Comparator/tree/master/Image-Comparator-Dockerfiles)

Once setup you should have two things running:
* Web Server
* Couchdb instance

![Initial Setup](./readme_images/initial_setup.jpg)


To finish configuring a single node setup, run the following;
```
DB_PORT=5984
COUCHDB_USER=admin
COUCHDB_PASSWORD=password

curl -X PUT http://$COUCHDB_USER:$COUCHDB_PASSWORD@0.0.0.0:$DB_PORT/_users
curl -X PUT http://$COUCHDB_USER:$COUCHDB_PASSWORD@0.0.0.0:$DB_PORT/_replicator

# Use below if in admin party mode:
# curl -X PUT http://0.0.0.0:$DB_PORT/_users
# curl -X PUT http://0.0.0.0:$DB_PORT/_replicator
```

To create and setup the database, run the following:
```
DB_NAME=image_comparator 

curl -X PUT http://$COUCHDB_USER:$COUCHDB_PASSWORD@0.0.0.0:$DB_PORT/$DB_NAME

# Admin party:
# curl -X PUT http://0.0.0.0:$DB_PORT/$DB_NAME
```

Add some views to the db:
```
cd dbutil

curl -X PUT http://$COUCHDB_USER:$COUCHDB_PASSWORD@0.0.0.0:$DB_PORT/$DB_NAME/_design/basic_views -d @flask_server/image_comparator/utils/basic_views.json

# Admin party:
# curl -X PUT http://0.0.0.0:$DB_PORT/$DB_NAME/_design/basic_views -d @flask_server/image_comparator/utils/basic_views.json

cd ../ # return to main directory
```

### Add Images to DB:

```
docker exec -it -w $PWD image-comparator-flask bash
```

Place your imaging data in "Image-Comparator-Data". 

Ex:
```
root@a53f9696685a:/home/bb927/Image-Comparator# ls ../Image-Comparator-Data/
MR.1.dcm   MR.2.dcm  MR.3.dcm  MR.4.dcm
```


> If you want to supply the class of the image ahead of time, supply a csv in "Image-Comparator-Data" with image name and id like so:
```
image,class
MR.1.dcm,class_a
MR.2.dcm,class_b
MR.3.dcm,class_c
MR.4.dcm,class_a
...
```

Run addImages.py to add images:
```bash
python3 image_comparator/utils/addImages.py <path to Image-Comparator-Data> <imageSetName> [<fromCSV>]
```

Ex:
```bash
python3 flask_server/image_comparator/utils/addImages.py Image-Comparator-Data TEST
```
```bash
python3 flask_server/image_comparator/utils/addImages.py Image-Comparator-Data TEST test_classification.csv

python3 flask_server/image_comparator/utils/addImages.py Image-Comparator-Data mimicMIDRCtrain training_classification.csv
python3 flask_server/image_comparator/utils/addImages.py Image-Comparator-Data mimicMIDRCvalidation validation_classification.csv
python3 flask_server/image_comparator/utils/addImages.py Image-Comparator-Data mimicMIDRCtest test_classification.csv
```

* \<imageFolder> Image-Comparator-Data, but could be user specified.  
* \<imageSetName> is the name for the image set.  
* \<fromCSV> ```name of optional csv file in imageFolder```

### Change flask_server/app/templates/app_template.html to have your users in this section:

```html
<!-- manually add users that you assigned tasks too in makeTasks.rb -->
<select id="username" onchange="{{ app_config.app }}TaskFeeder.OnSetUser(this.value)">
    <option selected="selected">Choose your user</option>
    <option value="Benjamin">Benjamin</option>
</select>
```

*With the app running, image list created and users in the drop down, you can proceed to configuring image lists for use within specific apps below*

## Apps

> You need to have created an image set already with "addImages.py". Go back to the "Add Images to DB" section above.

> Shell into container if you haven't already and make your way to "./dbutil"
```
docker exec -it -w $PWD image-comparator-flask bash
```

### Image-Comparator

#### Make Image Compare List:

```bash
ruby makeICLFromImageSetName.rb <imageSetName> <pct repeat> <list name>
```

```bash
python3 flask_server/image_comparator/utils/makeCompareList.py <imageSetName> <list name> <pct repeat>
```

Ex:
```bash
python3 flask_server/image_comparator/utils/makeCompareList.py TEST TESTCompareList
```

```bash
python3 flask_server/image_comparator/utils/makeCompareList.py TEST TESTCompareList 10
```

Ex:
```
ruby makeICLFromImageSetName.rb <imageSetName> <pct repeat> <list name>
ruby makeICLFromImageSetName.rb <imageSetName> <pct repeat> <list name>
ruby makeICLFromImageSetName.rb <imageSetName> <pct repeat> <list name>
```

* \<imageSetName> is the same name that was given to addImagesToDb.rb. This is you image set.  
* \<pct repeat> is the percentage of repeated pairs to be displayed.  
* \<list name> is a new Image Compare List name.  

#### Add a task to a user:

```bash
python3 makeTask.py <user> <image-list-name> <image-list-type> <task-order> [<description>]
```

```bash
python3 flask_server/image_comparator/utils/makeTask.py Benjamin TESTCompareList compare 1

python3 flask_server/image_comparator/utils/makeTask.py Benjamin TESTCompareList compare 1 test_description
```

* \<user> is who should complete the task  
* \<list name> <list name> from above in (makeICLFromImageSetName.rb)  
* \<image-list-type> compare, OCTcompare, classify, classify_nine or quandrant.  
* \<task-order> is what precedence the image compare task takes relative to others assigned to this user.  


### Image-Classifier

#### Make Image Classify List
```bash
ruby makeImageClassifyList.rb <imageSet> <listName> <pctRepeat>
```

Ex:
```bash
python3 flask_server/image_comparator/utils/makeClassifyList.py TEST TESTClassifyList
```

```bash
python3 flask_server/image_comparator/utils/makeClassifyList.py TEST TESTClassifyList 10
```

#### Add a task to a user:
```bash
python3 makeTask.py <user> <image-list-name> <image-list-type> <task-order> [<description>]
```

```bash
python3 flask_server/image_comparator/utils/makeTask.py Benjamin TESTClassifyList classify 1

python3 flask_server/image_comparator/utils/makeTask.py Benjamin TESTClassifyList classify 1 test_desc
```

### Grid-Classifier

#### Make Image Grid List
```bash
python3 flask_server/image_comparator/utils/makeGridList.py <imageSet> <listName>
```

```bash
python3 flask_server/image_comparator/utils/makeGridList.py TEST TESTGridList
```

#### Add a task to a user:
```bash
python3 flask_server/image_comparator/utils/makeTask.py <user> <image-list-name> <image-list-type> <task-order> [<description>]
```

```bash
python3 flask_server/image_comparator/utils/makeTask.py Benjamin TESTGridList grid 1

python3 flask_server/image_comparator/utils/makeTask.py Benjamin TESTGridList grid 1 test_description
```

### MIDRC-Challenge0

#### Make Challenge-0 List
```bash
python3 flask_server/image_comparator/utils/makeMIDRCChallenge0List.py <imageSet> <listName>
```

Ex:
```

```

#### Add a task to a user:
```bash
python3 flask_server/image_comparator/utils/makeTask.py <user> <image-list-name> <image-list-type> <task-order> [<description>]
```

Ex:
```

```


## Acknowledgements

In the order they appear this project has been forked and added to. Yet again we fork to make a QTIM-Lab based project that will be adapted and maintained here.

1. Jayashree Kalpathy-Cramer, PhD for original source code (https://github.com/AlanCramer/Image-Comparator)  
2. Collin Wen (https://github.com/CollinWen/Image-Comparator.git)  
3. Collin Wen (https://github.com/CollinWen/Image-Comparator-Dockerfile)  
4. Jimmy chen (https://github.com/jche253/Fundus_Classifier_Comparator)  


## Contact
For questions, please contact:
* Benjamin Bearce, bbearce@gmail.com  
* Jayashree Kalpathy-Cramer, PhD kalpathy@nmr.mgh.harvard.edu  

