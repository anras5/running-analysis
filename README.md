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
