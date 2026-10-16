# swcpy software development kit (SDK)
This is the python SDK to to interact with the SportsWorldCentral Football API, which was created for the book [Hands-On APIs for AI and Data Science](https://handsonapibook.com).

## Installing swcpy

To install this SDK in your environment, execute the following command:

`pip install swcpy@git+https://github.com/{owner of repo}/portfolio-project#subdirectory=sdk`

## Example Usage

This SDK implements all the endpoints in the SWC API, in addition to providing bulk downloads of the SWC fantasy data in CSV format.

### Setting base URL for the API
The SDK looks for a value of `SWC_API_BASE_URL` in the environment. The preferred method for setting the base URL for the SWC API is by creating a Python `.env` file in your project directory with the following value:

```
SWC_API_BASE_URL={URL of your API}
```

You may also set this value as an environment variable in the environment you are using the SDK, or pass it as a parameter to the `SWCConfig()` method.


### Example of normal API functions

To call the SDK functions for normal API endpoints, here is an example:

```python
from swcpy import SWCClient
from swcpy import SWCConfig

config = SWCConfig(swc_base_url="http://0.0.0.0:8000",backoff=False)
client = SWCClient(config)    
leagues_response = client.list_leagues()
print(leagues_response)
```

### Data Validation and Pydantic Models
All data returned by this SDK is parsed into **Pydantic model objects** (for example, `League`, `Player`, `Team`, and `Performance`).  
This means:

- Returned objects are **type-safe and structured** — you can access fields directly as attributes (e.g., `player.first_name` instead of `player["first_name"]`).  
- The SDK automatically **validates** all response data against the expected schema.  
- If the API returns malformed or incompatible data, a `pydantic.ValidationError` will be raised, ensuring that only valid and well-formed data is passed to your code.

#### Example: Using returned Pydantic objects
```python
players = client.list_players()

for p in players:
    print(f"{p.first_name} {p.last_name} - Position: {p.position}")
```

### Example of bulk data functions

The build data endpoint return a bytes object. Here is an example of saving a file locally from a bulk file endpoint:

```python
import csv
import os
from io import StringIO

config = SWCConfig()
    client = SWCClient(config)    

    """Tests bulk player download through SDK"""
    player_file = client.get_bulk_player_file()


    # Write the file to disk to verify file download
    output_file_path = data_dir + 'players_file.csv'
    with open(output_file_path, 'wb') as f:
        f.write(player_file)
```

# Configuring Retry Behavior

The SDK supports automatic retry with **progressive backoff with jitter**, which retries failed API calls (such as temporary network errors or rate limits) for a limited period before giving up.  
This behavior is configured through the `SWCConfig` object.

```python
from swcpy import SWCConfig, SWCClient

# Example: enable backoff with a maximum retry time of 30 seconds
config = SWCConfig(
    backoff=True,
    backoff_max_time=30
)

client = SWCClient(config)
```
By default, retries are **enabled** with exponential backoff up to **30 seconds**.  
You can disable them by passing `backoff=False` if you prefer to handle retry logic manually.



