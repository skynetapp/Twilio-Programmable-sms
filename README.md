#### Date: 25-11-2016
#### Description: Twilio programmable sms

#### The flow is as follows:

#### Step 1:
  Login with credentials.
  
#### Step 2:
   Goto programmable sms.
   
   Link : https://www.twilio.com/console/sms/dashboard
  
#### Step 3:
   Create messaging service.
   
   Ex: Test 1
  
#### Step 4:
  In Configure In Inbound Settings set REQUEST URL.
  
  Ex: http://159.203.239.91/twilio1.0/requestHandler.php
  
#### Step 5:
  * Request  : To this url the twilio server will send a request containing contactnumber, message, country code etc.
  * Response : This file should return proper xml formated output.
  
#### Step 6:
  In Numbers we can add contacts to which the sms can be delivered.
  
#### Step 7:
  The received messages and reply if any errors can be found in the log.
  
  Link : https://www.twilio.com/console/sms/dashboard
