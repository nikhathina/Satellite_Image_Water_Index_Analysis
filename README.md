import rasterio
import numpy as np

swir_path = r"yourpath.tiff"  # NIR band path
B3_path = r"E:\yourpath.tiff"  # Green band path

def calculate_ndwi(B3_path, swir_path, output_path):
    with rasterio.open(swir_path) as nir, rasterio.open(B3_path) as green:
        nir_band = nir.read(1).astype("float32")
        green_band = green.read(1).astype("float32")

        # Handle no-data values in both bands (replace -9999 or other no-data value if needed)
        nir_nodata = nir.nodata  # Get the no-data value from the NIR band (if any)
        green_nodata = green.nodata  # Get the no-data value from the Green band (if any)

        # Mask no-data values
        if nir_nodata is not None:
            nir_band[nir_band == nir_nodata] = np.nan
        if green_nodata is not None:
            green_band[green_band == green_nodata] = np.nan

        # Calculate NDWI
        ndwi = (green_band - nir_band) / (green_band + nir_band)

        # Remove NaN values (optional: you can set NaN to 0 or another placeholder value)
        ndwi[np.isnan(ndwi)] = 0  # Replace NaNs with 0 (or use a different value if necessary)

        # Ensure the metadata for the output TIFF is properly updated
        meta = nir.meta
        meta.update(dtype='float32', count=1)

        # Write the NDWI to the output TIFF
        with rasterio.open(output_path, "w", **meta) as dst:
            dst.write(ndwi, 1)

# Replace with actual file paths
calculate_ndwi(swir_path, B3_path, "NDWI_Output.tif")

# Reading the NDWI output file for visualization
with rasterio.open("NDWI_Output.tif") as src:
    ndwi_data = src.read(1)  # Read the NDWI data from the first band

import matplotlib.pyplot as plt
from matplotlib.colors import LinearSegmentedColormap
import seaborn as sns
# Checking the histogram of NDWI values to understand the range
plt.figure(figsize=(8, 6))
plt.hist(ndwi_data.flatten(), bins=50, color='blue', alpha=0.7)
plt.title("Histogram of NDWI Values")
plt.xlabel("NDWI Value")
plt.ylabel("Frequency")
plt.grid(True)
plt.show()

# Custom colormap: blue for water and green for vegetation
colors = [(0, "blue"), (0.5, "white"), (1, "green")]  # Blue -> White -> Green
cmap = LinearSegmentedColormap.from_list("BlueGreen", colors)

# Visualization of NDWI
plt.figure(figsize=(10, 6))
sns.set(style="whitegrid")  # Optional: for styling the plot

# Clip the NDWI values to the range -1 to 1 for better visualization
ndwi_data_clipped = np.clip(ndwi_data, -1, 1)

# Plot NDWI with the custom colormap
plt.imshow(ndwi_data, cmap=cmap, vmin= -1, vmax= 1)  # Adjust vmin and vmax to match NDWI range
plt.colorbar(label='NDWI')
plt.title("NDWI Visualization (Water: Blue, Vegetation: Green)")
plt.show()

import rasterio
import numpy as np
import matplotlib.pyplot as plt

def plot_ndwi(ndwi_path):
    with rasterio.open(ndwi_path) as src:
        ndwi = src.read(1)

    # Remove NaN or invalid values
    ndwi = np.where(np.isnan(ndwi), 0, ndwi)

    # Auto-adjust vmin and vmax using percentiles
    vmin = np.nanpercentile(ndwi, 2)  # 2nd percentile
    vmax = np.nanpercentile(ndwi, 98) # 98th percentile

    # Plot NDWI
    plt.figure(figsize=(10, 6))
    plt.imshow(ndwi, cmap="BrBG", vmin=vmin, vmax=vmax)
    plt.colorbar(label="NDWI")
    plt.title(f"NDWI Visualization\n(vmin={vmin:.2f}, vmax={vmax:.2f})")
    plt.show()

# Call the function
plot_ndwi("NDWI_Output.tif")
