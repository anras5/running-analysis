# Running Analysis

This repository contains code for clustering runs. The main idea is that we:
1. Get data from Polar Flow app
2. Upload the data into `polar-data` directory.
3. Run the `import.ipynb` notebook that puts the data into MongoDB.
4. Get the clustering using one of the notebooks from `notebooks` directory.

### Approaches

##### 1
The first approach (the first idea and also suggested by Gemini) was to use the raw data from GPS. So:
1. We get the data from MongoDB for every run.
2. For each run, we calculate the distance between consecutive points.
3. For each run, we normalize the length of the run (because longer runs will have more points) by interpolation. This part actually works pretty well, I did not expect it tbh - we can go from many many points to around 70 points for a 10 km run and not lose too much information. I checked in on a map.
4. We also need to get rid of outliers (because it sometimes happened that I ran while on vacation and the coordinates were very different from the rest ). For this I used IsolationForest.
5. We then cluster the runs using KMeans.

Results: I don't really like them - they seem kinda random. 10km and 3km runs are in the same clusters. Some runs that have identical paths end up in different clusters (this is what I don't like the most).

#### 2
The second approach is something even more simple than the first one. The main idea is to get general information about the run: distance, maximum and minimum coordinates.
1. We get the data from MongoDB for every run.
2. For each run, we get the distance, maximum and minimum coordinates.
3. We get rid of the outliers (similar way to the first approach).
4. We then cluster the runs using KMeans.

Results: Still not great, maybe even worse than the first approach.

#### 3
The third approach takes on the first one but with a different approach. This time instead of passing the raw coordinates (after interpolation) of the runs, we calculate the "distance" between runs by ourselves and then use DBSCAN. This is because with the first approach, it seemed like the algorithm for clustering was overwhelmed by the number of points, and all the tested dimensionality reduction techniques did not help much.

1. We get the data from MongoDB for every run.
2. For each run, we calculate the distance between consecutive points.
3. For each run, we normalize the length of the run (because longer runs will have more points) by interpolation to 50 points (more points mean much longer computation time).
4. For each pair of runs, we calculate "distance" between them. To do this, we use the Dynamic Time Warping (DTW) algorithm - (`compute_distance` function).
5. We then cluster the runs using DBSCAN with a precomputed distance matrix.

The results look very good, finally something like I expected. More work could be done in regards to the length of the runs, because now we need to provide the `eps` parameter for DBSCAN. However, for longer runs (even similar ones), it's normal that the "distance" is going to be higher because of the increased length of the run (even though the runs are interpolated).
