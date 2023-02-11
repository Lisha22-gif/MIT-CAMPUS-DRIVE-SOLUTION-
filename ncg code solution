from flask import Flask, jsonify, request
import requests
import hashlib
import time

app = Flask(__name__)

# AQICN API endpoint
AQICN_API_URL = "https://api.waqi.info/search/?token=<API_KEY>&keyword={}"

# Cache storage
cache = {}

# Max cache entries
MAX_CACHE_SIZE = 100

# Cache expiry time in seconds
CACHE_EXPIRY = 3600

def fetch_aqi_from_api(city):
    url = AQICN_API_URL.format(city)
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        return None

def get_aqi(city):
    # Hash the city name for cache key
    cache_key = hashlib.sha256(city.encode()).hexdigest()

    # Check if the key is already in cache
    if cache_key in cache:
        cache_entry = cache[cache_key]
        # Check if cache entry is expired
        if time.time() - cache_entry["timestamp"] < CACHE_EXPIRY:
            return cache_entry["data"]
        else:
            # Delete the expired cache entry
            del cache[cache_key]

    # Fetch AQI from the API if not in cache
    aqi = fetch_aqi_from_api(city)
    if aqi:
        # Add the AQI to cache if there is space
        if len(cache) < MAX_CACHE_SIZE:
            cache[cache_key] = {
                "timestamp": time.time(),
                "data": aqi
            }

    return aqi

@app.route("/aqi", methods=["GET"])
def get_aqi_api():
    city = request.args.get("city")
    if not city:
        return "Error: Missing city parameter", 400

    aqi = get_aqi(city)
    if aqi:
        return jsonify(aqi)
    else:
        return "Error fetching AQI from API", 500

if __name__ == "__main__":
    app.run()
