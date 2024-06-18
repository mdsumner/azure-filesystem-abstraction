## sigh

This is my first reponse to this blog post, I really appreciate the details there as it gives me something to go on. But also, how ridiculous is this azure/python/json stuff - a json api is fine, you can read text from the internet with a query - why all this extra layers? 

https://blog.rtwilson.com/accessing-planetary-computer-stac-files-in-duckdb/

Why all this silliness? Well I feel that the python/stac/azure/goog/aws tower is a little bit too much inferno now. STAC is text from the internet, parse it into a dataset and traverse the tree. Find URLs in it. Group and schedule how you like.  Authenticate. Stream the data, we have generic tools for this, in many programming langs. 

Here's the Parquet in a straightforward URL: 

```
https://github.com/mdsumner/azure-silliness/raw/main/io-lulc-9-class.parquet
```

Here's how I got access to this Parquet, and save it here so we can use a straightfoward URL. 

```python3
from pystac_client import Client
import planetary_computer
import warnings
warnings.simplefilter(action='ignore', category=[FutureWarning, UserWarning])

import geopandas
catalog = Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1/",
    modifier=planetary_computer.sign_inplace,
)

asset = catalog.get_collection("io-lulc-9-class").assets["geoparquet-items"]

df = geopandas.read_parquet(
    asset.href, storage_options=asset.extra_fields["table:storage_options"]
)
df.to_parquet("io-lulc-9-class.parquet")

## now things I tried for vsiaz
#export VSICURL_PC_URL_SIGNING=YES
#export AZURE_STORAGE_ACCOUNT=pcstacitems
#export AZURE_STORAGE_SAS_TOKEN='st=202...<insert your own here>'
#(AZURE_STORAGE_ACCESS_KEY or AZURE_STORAGE_SAS_TOKEN or AZURE_NO_SIGN_REQUEST) 
#ogrinfo /vsiaz/items/io-lulc-9-class.parquet
```
