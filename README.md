#### Date: 25-11-2016
#### Description: Twilio programmable sms

#### The Folder Structure is as follows:
   
   
   Root Directory | Sub Directory
------------ | ------------- 
index.php | | |
Global | DBmysql(MySQL Connection)  |
Lib | AlchemyAPI |
Modules | Twilio | Twilio Controller, Twilio Action, Twilio MySql


#### The flow is as follows:

#### Step 1:
  Login with your registered Email and Password credentials.
 
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
  Request  : To this url the twilio server will send a request containing contactnumber, message, country code etc.
  
  Function **requestProcessor** will be called in Controller. requestHandler.php -> Modules -> Twilio -> TwilioController.php
  
  **_Code:_**
	
```
   public function requestProcessor($post_data){
    
	$data= new DBMySQL();
	$connection=$data->getConnection();
    $data= new TwilioMysql();
	$data->setTwilioMySQLConnection($connection);
	$data->insertIntoTwilioRequest($post_data);
    
	// parsing alchemy api  
    $twilio_action = TwilioAction::getInstance();
	$result = $twilio_action->twilioAlchemyProcessorAction($post_data); //create value object with listing parameters if any.
			
    }
```
  
#### Step 6:
   In Class **TwilioMySQL**, **insertIntoTwilioRequest($post_data)** will be called to insert the message into table.
  
    **_Code:_**
	
```
   public function insertIntoTwilioRequest($input_arr){
        $body=$input_arr['Body'];
		$from=$input_arr['From'];
		$sql = "INSERT INTO twilio_request
				(
				request_contact,
				request_message,
				send_to_alchemy
				) VALUES (
				'$from',
				'$body',
				'1'
				)";
		mysqli_query($this->con,$sql);
       // return $this->con->insert_id;
    }
 ```
     
#### Step 7:
   By Calling function **insertIntoRequest** the message request is stored in Request table and returns the inserted record Id.
   
  **_Code:_**
	
```
function twilioAlchemyProcessorAction($input_arr)
   {
    $inputString1= trim($input_arr['Body']);
   	$inputString=strtolower($inputString1);
    $from=$input_arr['From'];
    $alchemy_apicall = new AlchemyAPI($GLOBALS['alchemy_apiKey']);
    $data = $alchemy_apicall->entities('text', $inputString, null);
    $dbFactory= new AlchemySqlManagement();
    $requestId=$dbFactory->insertIntoRequest('text','','','sms',$from);
    $masterId=$dbFactory->insertIntoMasterData($data,$requestId,$resume_key,$file_name); 
    
   
   }
   
function insertIntoRequest($return,$fileName,$fileExt,$resume_upload,$resumekey)
		{
			$sql = "INSERT INTO BlueMixAlmEntityExtractReq (text,file_name,file_ext,type,owner_id)
				VALUES ( '$return','$fileName','$fileExt','$resume_upload','$resumekey')";
			$res=$this->query($sql);
			$requestId=mysql_insert_id();
			return $requestId;
		} 
 ```
#### Step 8:
   The message request will be passed to Alchemy API response function **entities('text', $inputString, null)**.
  
#### Step 9:
   Function **insertIntoMasterData($data,$requestId,$resume_key,$file_name)** will be called to insert data into master table.
   **_Code:_**
	
```  
   function insertIntoMasterData($json_response,$id,$resume_key,$file_name)
		{
			$this->response_array =$json_response;
			$response=json_encode($json_response);
			echo $str="INSERT INTO alchemy_master
               (
                alm_sugar_id,
                alm_response_text,
                alm_date,alm_external_id,
				file_name,
				resume_key
                ) VALUES (
                
                '',
                '$response',
                NOW(),
				'$id',
				'$file_name',
				'$resume_key'
                )";
			$res=$this->query($str);
			
			$masterId=mysql_insert_id();
			
			if($masterId>0)
		return $this->getParsedDataFromJSONResponse($masterId);
		}
```
		
#### Step 10:
   Here based on the master request id, the response will be stored in Child table by function **getParsedDataFromJSONResponse($masterReqId)**.

**_Code:_**

```
   public function getParsedDataFromJSONResponse($masterId){
    	foreach($this->response_array['entities'] as $entityKey=>$entityVal) {
            $row['alc_master_id']=$masterId;
            $row['alc_type']=$entityVal['type'];
            $row['alc_relevance']=$entityVal['relevance'];
            $row['alc_count']=$entityVal['count'];
            $row['alc_text']=$entityVal['text'];
            //$row['alc_quotations_statement']=$entityVal['statement'];
            $row['alc_knowledgegraph_typehierarchy']=$entityVal['knowledgegraph']['typehierarchy'];
            $row['alc_sentiment_type']=$entityVal['sentiment']['type'];
            $row['alc_sentiment_score']=$entityVal['sentiment']['score'];
            $row['alc_sentiment_mixed']=$entityVal['sentiment']['mixed'];
            $row['alc_disambiguated_name']=$entityVal['disambiguated']['name'];

            $row['alc_disambiguated_website']=$entityVal['disambiguated']['website'];
            $row['alc_disambiguated_geo']=$entityVal['disambiguated']['geo'];
            $row['alc_disambiguated_dbpedia']=$entityVal['disambiguated']['dbpedia'];
            $row['alc_disambiguated_yago']=$entityVal['disambiguated']['yago'];
            $row['alc_disambiguated_opencyc']=$entityVal['disambiguated']['opencyc'];
            $row['alc_disambiguated_umbel']=$entityVal['disambiguated']['umbel'];
            $row['alc_disambiguated_freeebase']=$entityVal['disambiguated']['freeebase'];
            $row['alc_disambiguated_ciafactbook']=$entityVal['disambiguated']['ciafactbook'];
            $row['alc_disambiguated_census']=$entityVal['disambiguated']['census'];
            $row['alc_disambiguated_geonames']=$entityVal['disambiguated']['geonames'];
            $row['alc_disambiguated_musicbrainz']=$entityVal['disambiguated']['musicbrainz'];
            $row['alc_disambiguated_crunchbase']=$entityVal['disambiguated']['crunchbase'];
            /*$row['alc_subtype_id']=$entityVal[''];
            $row['alc_subtype_name']=$entityVal[''];*/
            $row['alc_quotation']=$entityVal['quotation'];

            if(count($entityVal['disambiguated']['subtype'])>0)
            foreach($entityVal['disambiguated']['subtype'] as $disStypeKey => $disStypeVal){
                $row['alc_disambiguated_subtype']=$disStypeVal;
                $this->insertIntoRowMysqlTable($row);
            }
            else $this->insertIntoRowMysqlTable($row);
        }
	    return true;
    }

    public function insertIntoRowMysqlTable($rowData=array()){
    	$sql = "INSERT INTO alchemy_child(
                    alc_master_id,
                    alc_type,
                    alc_relevance,
                    alc_count,
                    alc_text,
                    alc_quotations_statement,
                    alc_knowledgegraph_typehierarchy,
                    alc_sentiment_type,
                    alc_sentiment_score,
                    alc_sentiment_mixed,
                    alc_disambiguated_name,
                    alc_disambiguated_subtype,
                    alc_disambiguated_website,
                    alc_disambiguated_geo,
                    alc_disambiguated_dbpedia,
                    alc_disambiguated_yago,
                    alc_disambiguated_opencyc,
                    alc_disambiguated_umbel,
                    alc_disambiguated_freeebase,
                    alc_disambiguated_ciafactbook,
                    alc_disambiguated_census,
                    alc_disambiguated_geonames,
                    alc_disambiguated_musicbrainz,
                    alc_disambiguated_crunchbase,
                    alc_subtype_id,
                    alc_subtype_name,
                    alc_quotation,
                    alc_date
                ) VALUES (
                    '".$rowData['alc_master_id']."',
                    '".$rowData['alc_type']."',
                    '".$rowData['alc_relevance']."',
                    '".$rowData['alc_count']."',
                    '".$rowData['alc_text']."',
                    '".$rowData['alc_quotations_statement']."',
                    '".$rowData['alc_knowledgegraph_typehierarchy']."',
                    '".$rowData['alc_sentiment_type']."',
                    '".$rowData['alc_sentiment_score']."',
                    '".$rowData['alc_sentiment_mixed']."',
                    '".$rowData['alc_disambiguated_name']."',
                    '".$rowData['alc_disambiguated_subtype']."',
                    '".$rowData['alc_disambiguated_website']."',
                    '".$rowData['alc_disambiguated_geo']."',
                    '".$rowData['alc_disambiguated_dbpedia']."',
                    '".$rowData['alc_disambiguated_yago']."',
                    '".$rowData['alc_disambiguated_opencyc']."',
                    '".$rowData['alc_disambiguated_umbel']."',
                    '".$rowData['alc_disambiguated_freeebase']."',
                    '".$rowData['alc_disambiguated_ciafactbook']."',
                    '".$rowData['alc_disambiguated_census']."',
                    '".$rowData['alc_disambiguated_geonames']."',
                    '".$rowData['alc_disambiguated_musicbrainz']."',
                    '".$rowData['alc_disambiguated_crunchbase']."',
                    '".$rowData['alc_subtype_id']."',
                    '".$rowData['alc_subtype_name']."',
                    '".$rowData['alc_quotation']."',
                    NOW()
                )";
        mysqli_query($this->con,$sql);
        return $this->con->insert_id;
    }
    
```
  
#### Step 11:
  In Numbers we can add contacts to which the sms can be delivered.
  
#### Step 12:
  The received messages and reply if any errors can be found in the log.
  
  Link : https://www.twilio.com/console/sms/dashboard
