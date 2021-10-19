# Task 1: Create a bucket

## Using the google console 
 
Navigation menu->Cloud Storage->Browser->Create bucket
  
Provide PROJECT ID as a name for your bucket.
Provide REGION and ZONE as specified in the lab.


# Task 2: Create a pub sub topic

## Using cloud shell

gcloud pubsub topics create myTopic


# Task 3: Create the thumbnail Cloud Function

## Using the google console

Navigation menu->Cloud Functions->Create functions

Set trigger to Cloud storage
Set entry point to Thumbnail

Replace the index.js with the provided code for it 
and modify it as directed(replace topic in line 15).

Replace the package.json with the provided code.


# Task 4: Remove the previous cloud engineer
## Using the google console

Navigation menu-> IAM & Admin->IAM
Remove Project Viewer access for Username 2 by clicking the trashcan icon next to the role name.
Then click SAVE.