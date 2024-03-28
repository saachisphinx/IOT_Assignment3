# IOT_Assignment3
To make IoT System following  steps to be followed:
Step 1: Make ThingSpeak Channel
ThingSpeak is an IoT platform that allows you to collect, analyze, and act on data from IoT devices. To create a ThingSpeak channel:
1.	Sign in to ThingSpeak or create a new account if you don't have one.
2.	Navigate to the Channels section and click on "New Channel."
3.	Fill in the required information such as name, description, and field labels.
4.	Click "Save Channel" to create your ThingSpeak channel.
Step 2: Add MQTT Device
MQTT (Message Queuing Telemetry Transport) is a lightweight messaging protocol for small sensors and mobile devices. To add an MQTT device to your ThingSpeak channel:
1.	In your ThingSpeak account, navigate to the "Apps" section and click on "ThingHTTP."
2.	Create a new ThingHTTP by specifying a name and description.
3.	Configure the ThingHTTP to send data to your ThingSpeak channel using the MQTT protocol. You'll need to specify the MQTT broker details and authentication credentials.
4.	Save your ThingHTTP configuration.
Step 3: Write Wokwi Code and Create Connection
Wokwi is an online platform for simulating and testing Arduino code. To write Wokwi code and create a connection to ThingSpeak:
1.	Write your Arduino code using the Wokwi Arduino simulator. This code should include MQTT client libraries to publish data to the ThingSpeak MQTT broker.
2.	Configure the MQTT client in your code to connect to the ThingSpeak MQTT broker using the credentials provided in your ThingSpeak channel.
3.	Test your code in the Wokwi Arduino simulator to ensure it works as expected.
4.	Once you're satisfied with your code, you can upload it to your physical Arduino device.
Step 4: Analyze the Data using MATLAB Analysis
MATLAB offers powerful tools for analyzing and visualizing IoT data. To analyze the data received from your ThingSpeak channel:
1.	Use MATLAB's ThingSpeak support package to connect to your ThingSpeak channel.
2.	Retrieve the data from your channel using MATLAB commands.
3.	Analyze the data using MATLAB's built-in functions and visualization tools. You can perform tasks such as data filtering, trend analysis, anomaly detection, etc.
4.	Visualize the results using plots, graphs, and other visualization techniques provided by MATLAB.


Code of wokwi
import network
import time
import urandom
from umqtt.simple import MQTTClient

# ThingSpeak MQTT broker details
mqtt_client_id = "OSMAGzMLLh8cPC4wDB0tOgU"
mqtt_user = "OSMAGzMLLh8cPC4wDB0tOgU"
mqtt_password = "ene6Wle7359cO+fx6gagtfww"
mqtt_server = "mqtt3.thingspeak.com"
mqtt_port = 1883
mqtt_topic_temperature = "channels/486827/publish/fields/field1"
mqtt_topic_humidity = "channels/486827/publish/fields/field2"
mqtt_topic_co2 = "channels/486827/publish/fields/field3"

# Wi-Fi details
WIFI_SSID = "Wokwi-GUEST"
WIFI_PASSWORD = ""

# Historical data storage
historical_data = []

# Function to generate random sensor values
def generate_sensor_data():
    temperature = urandom.uniform(-50, 50)
    humidity = urandom.uniform(0, 100)
    # Ensure CO2 value is within the acceptable range (300 to 2000 ppm)
    co2 = urandom.uniform(300, 2000)
    return temperature, humidity, co2

# Function to publish data to ThingSpeak
def publish_to_thingspeak(temperature, humidity, co2):
    client = MQTTClient(mqtt_client_id, mqtt_server, user=mqtt_user, password=mqtt_password)
    client.connect()
    client.publish(mqtt_topic_temperature, str(temperature))
    client.publish(mqtt_topic_humidity, str(humidity))
    client.publish(mqtt_topic_co2, str(co2))
    client.disconnect()

# Connect to Wi-Fi
sta_if = network.WLAN(network.STA_IF)
sta_if.active(True)
sta_if.connect(WIFI_SSID, WIFI_PASSWORD)

# Wait for Wi-Fi connection
while not sta_if.isconnected():
    pass

print("Connected to Wi-Fi")

# Main loop to generate and publish sensor data
while True:
    temperature, humidity, co2 = generate_sensor_data()
    publish_to_thingspeak(temperature, humidity, co2)
   
 print("Published: Temperature={:.2f}C, Humidity={:.2f}%, CO2={:.2f}ppm".format(temperature, humidity, co2))
    historical_data.append((temperature, humidity, co2))  # Store historical data
    if len(historical_data) > 720:  # Approximately 5 hours with data every 5 seconds
        historical_data.pop(0)  # Remove oldest data point if exceeds 5 hours
    time.sleep(5)  # Adjust the delay as needed (Reduced to 5 seconds for faster data entry)

def get_last_five_hours_data(sensor_type):
    # Assuming sensor_type is one of 'temperature', 'humidity', or 'co2'
    sensor_index = {'temperature': 0, 'humidity': 1, 'co2': 2}[sensor_type]
    last_five_hours_data = [(data[sensor_index],) for data in historical_data[-720:]]
    return last_five_hours_data

# Example usage to retrieve last five hours data for temperature
last_five_hours_temperature = get_last_five_hours_data('temperature')
print("Last five hours temperature data:", last_five_hours_temperature)


MATLAB Code:
% Set your ThingSpeak channel ID and read API key
channelID = 2486827 ;
readAPIKey = 'DRMUN5G1XVMKR8NN';

% Get the current time and time five hours ago
currentTime = datetime('now', 'TimeZone', 'UTC');
fiveHoursAgo = currentTime - hours(5);

% Set up the ThingSpeak URL for fetching data
url = sprintf('https://api.thingspeak.com/channels/%d/feeds.json?api_key=%s&start=%s&end=%s', ...
              channelID, readAPIKey, datestr(fiveHoursAgo, 'yyyy-mm-ddTHH:MM:SSZ'), ...
              datestr(currentTime, 'yyyy-mm-ddTHH:MM:SSZ'));

% Fetch data from ThingSpeak
data = webread(url);

% Extract sensor data
if ~isempty(data.feeds)
    sensorData = [data.feeds.field1]; % Assuming the sensor data is in Field 1
    timestamps = datetime({data.feeds.created_at}, 'InputFormat', 'yyyy-MM-dd''T''HH:mm:ss''Z''', 'TimeZone', 'UTC');
    
    % Display sensor data
    disp('Sensor Data:');
    disp(sensorData);
    disp('Timestamps:');
    disp(timestamps);
else
    disp('No data found in the specified time range.');
end
 

MATLAB CODE
% Set your ThingSpeak channel ID and read API key
channelID = 2486827 ;
readAPIKey = 'DRMUN5G1XVMKR8NN';

% Get the current time and time five hours ago
currentTime = datetime('now', 'TimeZone', 'UTC');
fiveHoursAgo = currentTime - hours(5);

% Set up the ThingSpeak URL for fetching data
url = sprintf('https://api.thingspeak.com/channels/%d/feeds.json?api_key=%s&start=%s&end=%s', ...
              channelID, readAPIKey, datestr(fiveHoursAgo, 'yyyy-mm-ddTHH:MM:SSZ'), ...
              datestr(currentTime, 'yyyy-mm-ddTHH:MM:SSZ'));

% Fetch data from ThingSpeak
data = webread(url);

% Extract sensor data
if ~isempty(data.feeds)
    sensorData = [data.feeds.field1]; % Assuming the sensor data is in Field 1
    timestamps = datetime({data.feeds.created_at}, 'InputFormat', 'yyyy-MM-dd''T''HH:mm:ss''Z''', 'TimeZone', 'UTC');
    
    % Display sensor data
    disp('Sensor Data:');
    disp(sensorData);
    disp('Timestamps:');
    disp(timestamps);
else
    disp('No data found in the specified time range.');
end

Wowki link : https://wokwi.com/projects/393468723481009153
Thingspeak : https://thingspeak.com/channels/2486827/private_show
