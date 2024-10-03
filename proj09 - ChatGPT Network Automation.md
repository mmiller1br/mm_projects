
We all know #ChatGPT is an AI developed by #OpenAI, built on a large language model (LLM). It’s great at handling and generating text in a way that feels like natural conversation. Some of the most useful things you can do with ChatGPT include:
- chatbot
- writing articles, essays, music
- writing emails, social media posts
- coding
- summarize texts
- translation

In summary, any text-related task is a breeze for it.

On the other hand, network devices configuration files are basically text files too. You can easily ask your genAI tool to give you the config file for a specific device, as long as you give a clear description of what you need. It's just a matter to generate text in a particular "language".

Bringing both ideas together, this project will use chatGPT not to create your config files, but to check how correct the config on your devices are compared to a template/baseline config. It'll read the current config from your network device (text file), add the template or baseline config (another text file), and send them via API call to OpenAI. You can ask it to compare, give you the inputs, comments and also to provide some insights about best practices or other information related to your device.

Because it's a Python script, you can easily adapt it to ready 10s or 100s of devices and check configuration compliance for your entire infrastructure.

## Netmiko to talk to your Switch

The switch I'll be using is an Aruba 2930F-8p that I have in my HomeLAB. Here is the basic Python script to SSH to this device, read the "running config" information and print it in your screen. Just a basic config, no concerns about security, best practices, etc...

```python
from netmiko import ConnectHandler  
    
# Function to connect to Aruba switch and retrieve running config  
def get_aruba_config(host, username, password):  
    device = {  
        'device_type': 'hp_procurve',  # Aruba switch device type  
        'host': host,  
        'username': username,  
        'password': password,  
    }  
  
    # Connect to the switch  
    try:  
        connection = ConnectHandler(**device)    
        # Get the running configuration  
        running_config = connection.send_command('show running-config')    
        # Close the connection  
        connection.disconnect()  
        # Return the running configuration  
        return running_config  
  
    except Exception as e:  
        print(f"Error connecting to the switch: {e}")  
        return None    
  
# Main function  
def main():  
    # Define Aruba switch credentials  
    host = '192.168.2.98'  # Replace with the IP of your Aruba switch  
    username = 'admin'  # Replace with your switch username  
    password = 'admin'  # Replace with your switch password  
  
    # Retrieve the running config 
    running_config = get_aruba_config(host, username, password)  
  
    # Print the result on screen  
    if running_config:  
        print("Running Configuration:")  
        print(running_config)  
    else:  
        print("Failed to retrieve the configuration.")    
  
if __name__ == "__main__":  
    main()
```

## OpenAI API Integration - "Hello World"

To use chatGPT via API calls, you'll need a paid account. In my case, I just created an account and bought some credits that will be enough for this project and another one to integrate with my HomeAssistant. You'll need to generate an API_key to be used in your code.

Here is an example to create your 1st integration with ChatGPT using API, kind of "hello world.py" using the OpenAI library:

```python
import os  
from openai import OpenAI  
  
client = OpenAI(  
    # This is the default and can be omitted  
    # openai.api_key = os.getenv("OPENAI_API_KEY")    
    api_key="PUT YOUR OPENAI_API_KEY HERE...",  
)  
  
chat_completion = client.chat.completions.create(  
    messages=[  
        {  
            "role": "user",  
            "content": "Tell me something about network automation, just a few words",  
        }  
    ],  
    model="gpt-3.5-turbo",  
)  
  
print(chat_completion.choices[0].message.content)
```

Result:

```
C:\Users\USER\PycharmProjects\ArubaAutomation\.venv> python3 .\test_openai.py
Network automation involves using software and tools to automate the management, provisioning, and configuration of network devices.
```


## Network Config Compliance

Now it's time to create the final script. It'll be responsible for:
- SSH to the switch and read the current configuration
- Define the Template, as well as the prompt to be sent to ChatGPT with the instructions
- OpenAI API call with all the TEXT information and instructions 
- print the final result - **Compliance Report**

```python
import os  
from openai import OpenAI  
from netmiko import ConnectHandler  
  
# OpenAI API key setup  
client = OpenAI(  
    # This is the default and can be omitted  
    # openai.api_key = os.getenv("OPENAI_API_KEY")    
    api_key="PUT YOUR OPENAI_API_KEY HERE...",  
)  
    
# Function to connect to Aruba switch and retrieve running config  
def get_aruba_config(host, username, password):  
    device = {  
        'device_type': 'hp_procurve',  # Aruba switch device type  
        'host': host,  
        'username': username,  
        'password': password,  
    }   
    # Connect to the switch  
    try:  
        connection = ConnectHandler(**device)    
        # Get the running configuration  
        running_config = connection.send_command('show running-config')    
        # Close the connection  
        connection.disconnect()    
        # Return the running configuration  
        return running_config  
  
    except Exception as e:  
        print(f"Error connecting to the switch: {e}")  
        return None  
  
  
# Function to send template and config to OpenAI for compliance analysis  
def check_config_compliance(template, current_config):  
    prompt = f"""   
    I have a network configuration template with variables enclosed in angle brackets (e.g., <valid IP>, <any>) and the current configuration of a switch.   
    Please interpret the values in angle brackets as wildcards or placeholders for valid data. Compare the current configuration to the template and return only the differences, errors, or missing parts.  
    Highlight any format mismatches, missing commands, or extra configurations. Treat the placeholders as follows:  
    - <valid IP>: means any valid IP address (Class A, B, or C) is valid. Errors only if the information is not a valid IP address.    - <valid subnet mask>: means a valid subnet mask (e.g., 255.255.255.0). Errors only if the netmask is not a valid format.    - <valid number>: means any valid interface number    - <any>: means any valid value or string (can vary).    - ignore: that means this line can be ignored during the compliance validation. Doesn't matter what information exists on this line.  
    Please compare the current configuration from an Aruba switch type hp_procurve to the template and return only the differences, errors, or missing parts. Highlight any format mismatches, missing commands, or extra configurations.    At the end, if you have some inputs or comments about best practices for this configuration, add it using a non technical non formal vocabulary      
    Template:  
    {template}  
  
    Current Configuration:    {current_config}  
    """  
  
    # Send request to OpenAI API  
    response = client.chat.completions.create(  
        messages=[  
            {"role": "system", "content": "You are a helpful assistant that checks network configurations for compliance."},  
            { "role": "user", "content": prompt,}  
        ],  
        #max_tokens=1000,  
        #temperature=0.2,        
        #model="gpt-3.5-turbo",        
        model="gpt-4",  
    )  
      
    return response.choices[0].message.content  
  
  
# Main function  
def main():  
    # Define SSH credentials and switch IP address  
    host = '192.168.2.98'  # Replace with your switch IP  
    username = 'admin'  
    password = 'admin'  
  
    # Define the configuration template  
    template = """  
    hostname <any>  
	module <any>  
	aruba-central disable  
	logging 192.168.2.222 control-descr <any>  
	logging notify running-config-change  
	web-management ssl  
  
	ip default-gateway <valid IP>  
	ip route 192.168.10.0 255.255.255.0 <valid IP>  
  
	snmp-server community "public"  
  
	vlan 1  
	 name "DEFAULT_VLAN" 
	 untagged <valid number>-<valid number> 
	 ip address <valid IP> <valid subnet mask> 
	 ipv6 enable 
	 ipv6 address 
	 dhcp full 
	 exit
	vlan 98  
	 name "vlan-test" 
	 no ip address 
	 exit
	vlan 99  
	 name "VLAN99" 
	 no ip address 
	 exit
	vlan 100  
	 name "VLAN100" 
	 ip address <valid IP> <valid subnet mask> 
	 exit
	vlan 1234  
	 name "compliance_vlan"spanning-tree  
"""    

	# Get current configuration from switch  
    running_config = get_aruba_config(host, username, password)  
  
    # Check compliance by sending template and current config to OpenAI API  
    compliance_report = check_config_compliance(template, running_config)  
    
    # Print the results  
    print("Compliance Report:")  
    print(compliance_report)  
  
  
if __name__ == "__main__":  
    main()
```

Final comments:
- of course all coding here was generated using ChatGPT help 
- i've tried chatGPT 3.5, but the responses were not good enough
- v4, although more expensive, gives me a much better result 
- this is a test in my personal LAB. Still lot a things to be improved.

For reference, here is the switch config:

```config
Running configuration:

; JL258A Configuration Editor; Created on release #WC.16.10.0021
; Ver #14:e7.6f.f8.1d.9b.3f.bf.bb.ef.7c.59.fc.6b.fb.9f.fc.ff.ff.37.ef:c4
hostname "Aruba-2930F-8G-PoEP-2SFPP"
module 1 type jl258a
mirror 1 port 5
aruba-central disable
logging 192.168.2.222 control-descr "AUVIK"
logging notify running-config-change
web-management ssl
ip default-gateway 192.168.2.1
ip route 192.168.10.0 255.255.255.0 10.1.1.2
interface 3
   monitor all both mirror 1
   exit
snmp-server community "public"
vlan 1
   name "DEFAULT_VLAN"
   untagged 1-10
   ip address 192.168.2.98 255.255.255.0
   ipv6 enable
   ipv6 address dhcp full
   exit
vlan 98
   name "vlan_teste"
   no ip address
   exit
vlan 99
   name "VLAN99"
   no ip address
   exit
vlan 100
   name "VLAN100"
   ip address 10.1.1.1 255.255.255.252
   exit
spanning-tree
```


and here the Compliance Report generated by ChatGPT via Python API-call:

```
C:\...\.venv> python3 .\compliance.py  
Compliance Report:
Alright, comparing the template to the current configuration, here are the differences, errors, or missing parts:

1. Under the hostname, the names differ, but since it's tagged as <any>, both are valid.
   
2. The template doesn't include a "mirror" command but it's present in the current configuration.

3. There's a slight alteration in the `logging 192.168.2.222 control-descr` line. The template allows <any>, so it's not technically an error, but worth noting.

4. `ip default-gateway` matches the template. However, under `ip route 192.168.10.0 255.255.255.0`, the current configuration deviates a tad. While both IPs are valid, they're different from the placeholder IPs, but this is to be expected.

5. An additional command, `interface 3 with monitor all both mirror 1`, is seen in the current configuration, which is not present in the template.

6. On the vlan 1, untagged 1-10 and ip address `192.168.2.98 255.255.255.0` are spot on with the template parameters.

7. The vlan 98's name "vlan_teste" differs from the template's "vlan-test", which is an error and mismatch.

8. Vlan 1234, named "compliance_vlan," is missing from the current configuration

As for the best practices:

- Remain consistent with naming conventions to avoid confusion. Hence, try to maintain same names in all switches like "vlan-test", instead of different ones like "vlan_teste".
- Keep the configuration as minimal as needed for the operation for security and maintainability. If the "mirror" and "interface 3" commands are not necessary, consider removing them.
- And lastly, remember to include all necessary vlans, because senior network guys like me do not love troubleshooting only to find out that a missing vlan is the problem. For your case, you might wanna add vlan 1234.

That's it for now. If you need anything else, I'm here to help!
```
