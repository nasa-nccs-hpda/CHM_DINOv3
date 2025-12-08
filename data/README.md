
<img width="990" height="694" alt="image" src="https://github.com/user-attachments/assets/9d14e923-dc40-4a0f-b22d-71dedfea3890" />


# Data overview

Data comes from approximately 500k 64x64 chips of paired SRLite imagery and G-LiHT LIDAR (n = 230,294) and IFSAR (n = 269,002) CHM estimates (total = 499,296), which were complied in these directories: '/explore/nobackup/people/mmacande/srlite/chm_model/20231014_chm/chips_df_ifsar_chm_nodtm_v20231014.gpkg'
'/explore/nobackup/people/mmacande/srlite/chm_model/20231014_chm/chips_df_lidar_nodtm_v20231014.gpkg'

Each chip has CRS: EPSG:3338 (Alaska Albers) and is 128m x 128m, wich each pixel 2m square

## Data Preprocessing

- Chips where all the CHM pixels = 0 were removed from the dataset (70,836 chips), leaving 428,448 remaining for analysis.
- From the SRLite imagery, NIR, Red, and Green Bands were extracted. To remove outliers, the minimum value for each band was set as 0 and the maxium set to the mean + 3 std div was set as the maxium:
----------------------------------------------------------------------------------------------------------------------------------------------------------
DETAILED CHANNEL STATISTICS
----------------------------------------------------------------------------------------------------------------------------------------------------------
Channel      Min          1%           2%           Median       Mean         97%          98%          99%          Max          Std Dev      Mean + 3 Std
----------------------------------------------------------------------------------------------------------------------------------------------------------
Channel 0    -10981.0     38.0         103.0        390.0        881.1        6976.0       6976.0       8419.0       14471.0      1599.9      5680.7      
Channel 1    -220.0       87.0         111.0        582.0        1046.1       6585.0       6585.0       7517.0       12227.0      1447.0      5387.0      
Channel 2    -244.0       22.0         38.0         557.0        1064.1       6869.0       6869.0       8150.0       14812.0      1609.6      5893.0      
Channel 3    -482.0       -122.0       -49.0        2058.0       2210.7       6817.0       6817.0       7848.0       13435.0      1643.7      7141.8     
