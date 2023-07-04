![[proj06_image1.png]]


# Network Automation with Voice Recognition
🎤🤖🚀

#raspberry #network #automation #ansible

## Description:

Imagine you, a Network Engineer, using your Google Assistant to send commands to you devices for you. In that case, you can talk to your Google assistant and ask it to do all the configuration that you need, without even touch your keyboard.

That's the idea behind this LAB !

What we need to make it real is to put some pieces together and make you Raspberry works for you:

- [Raspberry](https://www.raspberrypi.org/) PI (for this LAB, Raspberry PI version 3B)

- install [ANSIBLE](https://docs.ansible.com/ansible/latest/network/getting_started/index.html) on your Raspberry and test it with your Network device. Make sure it an access your device, connect to it, apply configurations, etc.

- install GOOGLE ASSISTANT on your Raspberry as well ... here you have a lot of tutorials on Youtube or, if you prefer, on Google SDK website ([https://developers.google.com/assistant/sdk/guides/library/python](https://developers.google.com/assistant/sdk/guides/library/python)). Make sure your Google Assistant is running OK before continue.

- [Python](https://www.python.org/) !!! Time to start coding ... Python is the engine responsible to put everything together. Changing the code from GoogleAssistant, creating your own script, etc, it will get the information from Google, manipulate it accordingly and call your Ansible playbook.

As you can see in this video, after everything is synchronized, all you need to do is talk to Google and ask her/him to do the work for you. Even my 10y old daughter can configure my switch now and create new VLANs, with NO physical interaction with any device.

Have fun, and share it !

Cheers. 🍻


## Demo Video:

[![Watch the video](https://youtu.be/NENi-g-LSzc/default.jpg)](https://youtu.be/NENi-g-LSzc)


## Reference

So, let me try to create a step-by-step. It's not gonna be a tutorial, but only a list of tasks if you want to do the same and have fun with your Network Automation project ...

​
1. Raspberry PI with Raspbian Linux installed (easy peasy, just follow the instructions [here](https://www.raspberrypi.org/documentation/installation/installing-images/), or search on Youtube some tutorial. There are a bunch ...)
    
2. Ansible installation... maybe a bit tricky, but in general you will need to run the following commands:

```bash
sudo apt-get update   
sudo apt-get upgrade  
sudo apt install ansible  
```   

After that, spend some time to play with Ansible and put it to work correctly, manually. For this LAB, I'm using an Aruba 2930 switch. It's not my intent to teach how to use Ansible, so let's assume that it's working and you already know how to play with playbooks, roles, inventories, etc ... Lots of material available, but I would say that Ansible DOCs is still the best place to get more information.

3. GoogleAssistant... there is a step-by-step at Google website, but if you make a quick search on Youtube, you'll probably find a good tutorial to do this part. Tricky part here is to find a good and compatible microphone/speaker. 


The main file used to run GoogleAssistant on your Raspberry PI is called pushtotalk.py. Find this file once we will need to make some changes on this file. I will try to explain how I did it ... just remembering this is a LAB, and my main objective here is to show it's possible ... fell free to make it better and share with others ;). 

After you have everything installed, it's time to start putting everything working together ...

1st thing 1st: create an **ACTION for your GoogleAssistant**:
here you will give a name for your Action (in my LAB I called it "Network Assistant"), and configure the phrases they will recognize, the "magic words" I need to say, and also the parameters it will understand (VLAN ID  $number) and send back to me as a response.

Tools from Google that you need to create this Action:

- Google Console: [https://console.actions.google.com/](https://console.actions.google.com/)

- Google DialogFlow: [https://dialogflow.cloud.google.com/#/login](https://dialogflow.cloud.google.com/#/login)

If you need a basic tutorial to create your first Action, check this Youtube here  :  
[Creating an Action for the Google Assistant from scratch](https://www.youtube.com/watch?v=plr65MD-FBY)

​
For your reference, in my LAB I created the following "Training Phrases":

![[proj06_image2.png]]


These are the phrases recognized by my Action. As you can see, numbers are recognized as variables, and you can use it as part of your responses for you Google Assistant. For your reference, below you can see the phrases I've used for responses:

![[proj06_image3.png]]


As you can see, the value $number is returned as part of my list of Responses.

After you create your ACTION, you can test it on your RaspberryPI + GoogleAssistant. Call your action, send one of the training phrases, and wait for a response. Check to see if Google is sending the parameter $number correctly as part of the response.

Next step, edit/modify the script pushtotalk.py on your Raspberry+GoogleAssistant. If you want to see the content of this file, you can check this link at Github:

[https://github.com/googlesamples/assistant-sdk-python/blob/master/google-assistant-sdk/googlesamples/assistant/grpc/pushtotalk.py](https://github.com/googlesamples/assistant-sdk-python/blob/master/google-assistant-sdk/googlesamples/assistant/grpc/pushtotalk.py)

Long story short, here is the portion of this file that I modified to call an external Python script (check the portion between "###")

``` python
        # This generator yields AssistResponse proto messages  
        # received from the gRPC Google Assistant API.  
        for resp in self.assistant.Assist(iter_log_assist_requests(),  
                                          self.deadline):  
            assistant_helpers.log_assist_response_without_audio(resp)  
            if resp.event_type == END_OF_UTTERANCE:  
                logging.info('End of audio request detected.')  
                logging.info('Stopping recording.')  
                self.conversation_stream.stop_recording()  
            if resp.speech_results:  
                logging.info('Transcript of user request: "%s".',  
                             ' '.join(r.transcript  
                                      for r in resp.speech_results))  
            ####  
            # check response, if contains "vlan" call a PYTHON script with response as TEXT  
            ####  
            if resp.dialog_state_out.conversation_state:  
               if resp.dialog_state_out.supplemental_display_text:  
                  # print the TEXT response  
                  logging.info(resp.dialog_state_out.supplemental_display_text)  
                  respstr = str(resp.dialog_state_out.supplemental_display_text)  
                  if (respstr.find('vlan') > 0):  ## check if response contains word "vlan"  
                     # PYTHON script with the string, create VAR file and CALL ANSIBLE playbook  
                     os.system("python /home/pi/env/lib/python3.5/site-packages/googlesamples/assistant/grpc/assistant_to_ansible.py %s" %respstr)  
            ###  
            ###
```


First, there is a variable called "resp.dialog_state_out.supplemental_display_text", and it has the response from your GoogleAssistant in a text string ! Checking when it contains the sub-string "vlan" is the trigger to call my external script called assistant_to_ansible.py !!

Next, lets see what we have in this script I've created:

```python
      import sys  
      import os

      args = sys.argv[1:]  
      ## open the FILE  
      f = open('/home/pi/.ansible/shana/ansible/roles/aruba-sw-vlan/vars/main.yml', 'w')

      if args:  
         #print(args)  ## display the string received from pushtptalk.py

         ## find the VLAN ID in the string from Google Assistant  
         index = args.index("vlan")  ## check the INDEX of the word 'vlan'  
         vlan_id = args[index+1]     ## next index is the VLAN ID  
         #print("VLAN = %s" %vlan_id) ## display the VLAN ID (debugging)

         ## mount the string and write the VARS file to be used by ANSIBLE  
         linetxt = 'vlan_id: ' + vlan_id + '\n'  
         print(linetxt)  
         f.write(linetxt)  
         linetxt = 'vlan_name: vlan_' + vlan_id + '\n'  
         print(linetxt)  
         f.write(linetxt)

         f.close()

         ## call ANSIBLE that creates a new VLAN  
         os.system('ansible-playbook ./playbooks/aruba-create-vlan.yml -i ./../../inventory/mmm0084/network/hosts.ini')

      else:  
         print("where is the argument ???")

      ## close the FILE  
      f.close()
```


Long story short again ... this script receives the string from the main GoogleAssistant script with the response. Then it searches for the VLAN_ID and update the following file: ....../vars/main.yml - this is a file with variables used by my Ansible Playbook and Roles. The content of this file is:

```
    vlan_id: 10  
    vlan_name: vlan_10
```


Only 2 lines, with the VLAN_ID and VLAN_NAME.

After update this file, last STEP (finally) ..... run the ANSIBLE playbook with this line:  
os.system('ansible-playbook ./playbooks/aruba-create-vlan.yml -i ./../../inventory/mmm0084/network/hosts.ini')

And that's it !

Call your GoogleAction, send the command (VLAN_ID), and run your Ansible Playbook using the response from your Google Assistant ...


Have fun  !!
