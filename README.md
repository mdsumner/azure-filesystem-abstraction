## filesystem abstractions

This is my first reponse to this blog post, I really appreciate the details there as it gives me something to go on. But also, this azure/python/json stuff is a bit of a towering epic - a json api is fine, you can read text from the internet with a query - why all this extra layers?  And I was frustrated, but also on the verge of figuring a lot out. 

https://blog.rtwilson.com/accessing-planetary-computer-stac-files-in-duckdb/

STAC is text from the internet, parse it into a dataset and traverse the tree. Find URLs in it. Group and schedule how you like.  Authenticate. Stream the data, we have generic tools for this, in many programming langs. 

My big epiphany is now: oh, it's all object storage, they have a protocol:// and containers "{bucket}" and then objects. Each is slightly different but GDAL has shown how common they can all be. 
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
