# API algorithm to apply object detection model to terabyte size satellite images with 800% better performance and 8 times less resources usage
Suppose we train our object detection model to find swimming pools.

Algorithm of the API - model input size 224x224 pixel image
![algorithm](https://github.com/orhannurkan/API-algorithm-for-terabyte-size-images-/algorithm.jpg)
Input format of the API : {[latitude_of_top_left, longitude_of_top_left, latitude_of_right_bottom, longitude_of_right_bottom]}

## 1 - Gets Coordinates : 
API gets coordinates of the selected zone as parameters. For example (in the satellite image below), [50.47, 3.91, 50.43, 3.96] here the first two elements of the input list are latitude-longitude of the left top corner of the selected zone and the last two elements of the input list are latitude-longitude of the right bottom corner of the selected zone. 
![coordinates](https://github.com/orhannurkan/API-algorithm-for-terabyte-size-images-/coordinates.jpg)

## 2 - Calculates Zone Image Size and > 300 MB : 
Then it calculates image size for this zone and checks whether it is less than 300MB or not. If the image of the zone is more than 300MB we will divide it to 300MB sectors. We can set any limit instead of 300MB but the most important detail is to have 224x224 pictures in that sector.

## 3 - Calculation of cols-rows to divide large picture into sectors(300MB chunks) : 
When we divide the zone to sectors it could have at the last column and at the last row less than 300MB sectors that is why we add pixels to complete to 224x224 pixel images.
In the yellow picture on the left you see how to create sectors of a large region. Before querying the DB, large region image size calculation is made and sector coordinates list is made, and then image of each sector with 300 MB are read 1 by 1 from the DB.
In the purple image on the right you see how to complete a less than 300 MB sector; The last column and last row image segments of the sector are completed as 224x224 pixels. So the sector is still under 300 MB, but it has the pixels needed by the AI model.
![segments](https://github.com/orhannurkan/API-algorithm-for-terabyte-size-images-/segments.jpg)
If zone is greater than 300MB we process every sector in 2 loops. For every row and for every column we calculate sector coordinates then call the function below.

## 4 - Function (coordinates) : 
In this function we get coordinates of a sector or selected little zone(less than 300MB) by the parameters. Then we make a SQL query request to get image from the DB. We divide image to 224x224 chunks to be able to send to model to predict pools in that little image because our model is trained by 224x224 pixels images. There is two loops for every row and column for sending image to model for prediction and getting and building pool list and insert these found pools every 1000 pools(we can set another value) to the DB. After finishing two loops function will exit.
The Function in the algorithm
![Function](https://github.com/orhannurkan/API-algorithm-for-terabyte-size-images-/function.jpg)

## 5 - Merge pool parts (for %800 better performance and less resources usage): 
We predict pools from 224x244 pixel images, so sometimes a part of a pool may be inside this 224x224 pixel image. However, other parts of that pool may be inside different images. Instead of overlapping the 224x224 pixel images by shifting them by 50%, we create a separate list for the pool parts coming to the edges of the images and merge this list at the end. By checking the coordinates of the pool parts, the pool parts will be merged by the API if they are from the same coordinates / pool.
In this way it is up to 8 times faster, both the DB traffic is reduced, the AI model is called 8 times less, system resources are not used unnecessarily, a single pool is created from its own parts and then saved as a whole to the DB. 
![overlapping](https://github.com/orhannurkan/API-algorithm-for-terabyte-size-images-/overlapping.gif)
