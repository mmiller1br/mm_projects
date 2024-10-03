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

