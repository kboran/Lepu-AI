# AI open platform interface document

> **This document is only provided to service docking parties and may not be released to the outside world without permission**



- [AI open platform interface document](#ai open platform interface document)
    - [Version Revision Record](#Version Revision Record)
    - [1. Request information](#1-Request information)
    - [2. Interface information](#2-Interface information)
        - [2.1 Request dynamic ECG AI analysis] (#21-Request dynamic ECG AI analysis)
        - [2.2 Request Resting ECG AI Analysis] (#22-Request Resting ECG AI Analysis)
        - [2.3 Batch query of analysis status](#23-Batch query of analysis status)
        - [2.4 Get analysis results](#24-Get analysis results)
            - [2.4.1 Dynamic ECG analysis results](#241-Ambulative ECG analysis results)
            - [2.4.2 Resting ECG analysis results](#242-Resting ECG analysis results)
    -[APPENDIX](#APPENDIX)
        - [dynamic heartbeat](#dynamiccardiogram)
        - [static cardio](#staticcardio)
        - [Analysis Status](#Analysis Status)



## Version revision record

| Time | Version | Modifier | Modification instructions |
| -------------- | ---- | ------ | -------- |
| December 15, 2021 | V0.1 | Wang Jiang | Initial release |
| April 26, 2022 | v0.2 | Wang Jiang | Update the AI ​​analysis and signature report to return the JSON structure |
| April 27, 2022 | v0.3 | kevin | Update some fields |
| May 18, 2022 | v0.4 | kevin | Update some field descriptions |
| September 23, 2022 | v0.5 | kevin | Added a new resting ECG analysis interface and improved some document content |
| May 12, 2023 | v0.5 | kevin | Improve document description |


## 1. Request information

1. Interface address

   Test Host: `https://open.viatomtech.com.cn`

2. Basic definition of interface
   The input (output) parameters of the business interface in this document all refer to business parameters. When calling the interface, Headers must add public request headers. When parsing the response, they must be parsed according to the public output parameters.

3. Character encoding
   UTF-8

4. Public request header-Header

   | Parameter name | Whether it is required | Type (length) | Description |
   | ------------ | -------- | ------------ | --------------- -------------------------- |
   | secret | is | string | given by Lepu |
   | access-token | Yes | string | Given by Lepu, different services use different access_token |
   | language | yes | string | zh-CN / en-US |

5. Public access

   | Parameter name | Whether it is required | Type (length) | Description |
   | -------- | --------- | ------ | ----------------------- --------------- |
   | code | is | int | response code. 0: Success, others: Exception |
   | message | Yes | string | Response description (commonly used for prompts after errors) |
   | reason | Yes | string | Error details (mainly stack information after the error) |
   | data | is | object \| array | response content (please refer to the output parameters of the specific business interface), the default is object, when requesting list data, it is array |

## 2. Interface information

### 2.1 Request dynamic ECG AI analysis

  Interface address: POST `${host}/api/v1/ecg/analysis/request` \
  Content-Type: `multipart/form-data`

 **Request body**

 | Parameter name | Whether it is required | Type (length) | Description |
 | ------------ | -------- | ------------ | --------------- -------------------------------------------------- |
 | ecg_file | No | .txt/.dat | Original file, need not be uploaded, please refer to AI Analysis Interface ECG File Protocol |
 | analyze_file | Yes | .txt | Lepu standard single-channel ECG file, this file is preferred as the analysis file, refer to the AI ​​analysis interface ECG file protocol |
 | ecg_info | is | string | JSON string, see appendix ecgInfo |

 **Response Parameters**

 | Parameter name | Whether it is required | Type (length) | Description |
 | ----------- | -------- | ------------| --------------- ------ |
 | analysis_id | Yes | string | Analysis ID, used to request analysis results |

example:

```jsonc
{
 "code": 0,
 "data": {
  "analysis_id": "Mg==|OTA="
 },
 "message": "Request successful",
 "reason": ""
}
```

### 2.2 Request resting ECG AI analysis

  Interface address: POST `${host}/api/v1/resting_ecg/analysis/request` \
  Content-Type: `multipart/form-data`

 **Request body**

 | Parameter name | Whether it is required | Type (length) | Description |
 | ------------| -------- | ------------ | --------------- ---------- |
 | ecg_file | is | .xml | ECG file in HL7 format |
 | ecg_info | Yes | string | JSON string, see appendix Static ECG ecgInfo |

 **Response Parameters**

 | Parameter name | Whether it is required | Type (length) | Description |
 | ---------- | -------- | ------------ | ---------------- -------- |
 | analysis_id | Yes | string | Analysis ID, used to request analysis results |

example:

```jsonc
{
 "code": 0,
 "data": {
  "analysis_id": "OA==|MzUyMg=="
 },
 "message": "Request successful",
 "reason": ""
}
```

### 2.3 Analysis status batch query

- This interface is suitable for dynamic ECG and resting ECG. Please view the corresponding status value description in the appendix.
- Before calling the query analysis result interface, first call this interface to obtain the status, and then obtain the results after the analysis is successful.

  Interface address: POST `${host}/api/v1/ecg/analysis/batch_status/query` \
  Content-Type：`application/json` \

 **Request body**

 | Parameter name | Whether it is required | Type (length) | Description |
 | ---------- | -------- | ------------ | ---------------- ------ |
 | analysis_ids | Yes | array | Analysis ID array |

example:

```jsonc
{
 "analysis_ids": [
  "Mg==|Njg=",
  "Mg==|OTA="
 ]
}
```

 **Response Parameters**

 | Parameter name | Whether it is required | Type (length) | Description |
 | ------ | -------- | ------------ | ------------ |
 | | Yes | array | Analysis status list |

example:

```jsonc
{
    "code": 0,
    "message": "Request successful",
    "reason": "",
    "data": [
        {
            "analysis_id": "Mg==|Njg=",
            "analysis_status": "80001"
        },
        {
            "analysis_id": "Mg==|OTA=",
            "analysis_status": "80001"
        }
    ]
}
```

### 2.4 Get analysis results
>Before requesting this interface, please obtain the analysis status first, and then request after the analysis is completed.

  Interface address: POST `${host}/api/v1/ecg/analysis/result/query` \
  Content-Type: `application/json`

 **Request body**

 | Parameter name | Whether it is required | Type (length) | Description |
 | ---------- | -------- | ------------ | ---------------- -------- |
 | analysis_id | is | string | Analysis ID, JSON encoded string. |

**Dynamic ECG response parameters**

 | Parameter name | Whether it is required | Type (length) | Description |
 | --------------- | -------- | ------------ | ----------- ------------------ |
 | analysis_id | Yes | string | Analysis ID, used to request analysis results |
 | analysis_status | yes | string | analysis status |
 | analysis_result | Yes | string | AI analysis result, please see the appendix for specific fields |
 | report_url | Yes | string | Analysis report, delayed compared to AI analysis results |

#### 2.4.1 Dynamic ECG analysis results

- AI analysis results
    
   

    
expand/collapse



    
   


- Sign report results

    
   

    
expand/collapse



    
   

    
#### 2.4.2 Resting ECG analysis results

- AI analysis/signature report results

    
   

    
expand/collapse



    
   


## Appendix

### Dynamic ECG

1.ecgInfo

```jsonc
   {
        "user": {
            "name": "", // name, required
            "phone": "", // Mobile phone number, required
            "gender": "1", // Gender (1: male; 2: female)
            "birthday": "1991-08-13", // Birthday (yyyy-MM-dd)
            "id_number": "", // ID number, required for signature reports
            "height":170, // Height in cm, optional
            "weight":80 // Weight in kg, optional
            //Gender and birthday will affect the analysis results, please fill in the real data
        },
        "analysis_type": "1", // 1 short-range, 2 long-range
        "service_ability": 1, // 1 AI analysis, 2 doctor's signature report
        "access_token": "", // AccessToken corresponding to service capabilities
        "application_id": "xxx.xxx.xx", //Application id (application package name, pay attention to the format)
        "device": {
            "model": "ER1", //Model, find the corresponding model from the following device list (case sensitive)
            "band": "Lepu", //Brand
            "sn": "" //Device sn
        },
        "ecg": {
            "measure_time": "2022-08-01 10:25:00", //Start measuring time, support UTC format (i.e. 0 time zone time, such as 2022-10-01T10:30:00Z, pay attention to the correct local current time conversion) and local current time format (such as 2022-08-01 10:25:00),
            //Note: If you use UTC format, you need to add the corresponding local time zone to get the local time. If you use the latter format, it is not required. Different formats of the measure_time field will affect the time of data, events, etc. in reports and analysis results.
            //Please use it correctly according to actual needs
            "duration": 30, //The duration of data analysis, unit s, not less than 30, >=900 is long-range, otherwise it is short-range, and must match the value of the analysis_type field
            "sample_rate": 125, //Device sampling rate, such as 500
            "lead": "II" // Lead I/Lead II
        }
    }
```

2. Equipment list

   | band | model | sampling rate | leads | remarks |
   | ---- | ----- | ------|----|----------- |
   | Lepu | ER1 | 125hz | II | Single lead Holter recorder |
   | Lepu | ER2 | 125hz | II | Single lead ECG recorder |
   | Lepu | ER2-S | 125hz | II | Single lead ECG recorder |
   | Lepu | ECG body fat scale S1 | 125hz | II | ECG body fat scale S1 |
   | Lepu | BP2 | Storage data 250hz/Real-time data 125hz | II | ECG sphygmomanometer |
   | Lepu | BP2-Pro| Storage data 250hz/Real-time data 125hz | II | ECG sphygmomanometer wifi version |
   | Lepu | Watch W1 | | II | Lepu Watch W1 |
   | | Universal Device Type | If you use another brand of device, 
please use "Universal Device Type" 
or contact us | | |

   Note: The device-related fields in ecgInfo must be consistent with those provided in this list, otherwise the analysis may fail.



4. Description of analysis results

    | | key | Chinese description | Type | Remarks |
    | ---- | ---- | ------- | ---- | ---- |
    |Report return object| | |||
    | baseInfo| | | | Basic information |
    | | testId | Check id | string | Used to intercept fragment data on the consumer side |
    | | sample | sampling rate | int | used to intercept fragment data on the consumer side |
    | | analysisTime | analysis time | string ||
    | | analysisDuration | analysis duration | int | unit seconds |
    | | name | name | string | These basic information are subject to the patient information on the consumer side |
    | | sex | gender| string ||
    | | age | age | int ||
    | | createTime | record date | string ||
    | | startTime | Data start time | string | Will be affected by measure_time format |
    | | endTime | end time of data | string | will be affected by measure_time format |
    | hrInfo | | Heart rate statistics|||
    | | beatCount | total number of beats | int ||
    | | averageHeartRate | average heart rate | int ||
    | | maxHeartRate | Fastest (maximum) heart rate | int ||
    | | minHeartRate | Slowest (minimum) heart rate | int ||
    | | longRRPeriodAt | The longest RR interval occurrence time | string ||
    | | longRRPeriodCount | Long RR (interval) number | int ||
    | | maxLongRRPeriod | Maximum RR interval duration | int ||
    | | asystoleRRPeriodCount | Number of asystole | int ||
    | | abnormalBeatCount | Abnormal heartbeat count | int | Signature report |
    | | ventricularBeatCount | ventricular beat | int | signed report |
    | | atrialBeatCount | supraventricular heartbeat | int | signature report |
    | hrvInfo | | | | Heart rate variability statistics |
    | heart rate variability | sdnn | SDNN | double ||
    | | sdnnIndex | SDNN Index | double ||
    | | rmssd | RMSSD | double ||
    | | pnn50 | PNN50 | double ||
    | | triangularIndex | triangular index | double||
    | | hf | HF | double||
    | | lf | LF | double ||
    | | vlf | VLF | double ||
    | afInfo | | | | Atrial fibrillation and atrial flutter events|
    | | afBeatPercent | (atrial flutter) proportion of atrial fibrillation | float ||
    | ventricularInfo | | | | ventricular rhythm |
    | ventricular | ventricularBeatCount | total number of ventricular beats | int ||
    | | ventricularPermatureBeatCount | Number of premature ventricular beats (single ventricular premature beat) | int ||
    | | coupleVentricularPermatureCount | Paired ventricular premature beats (paired ventricular premature beats) | int ||
    | | ventricularBigeminyCount | ventricular bigeminy number | int ||
    | | ventricularTrigeminyCount | number of ventricular trigeminy | int ||
    | | ventricularTachycardiaCount | Number of ventricular tachycardia (number of ventricular tachycardia occurrences) | int ||
    | | longestVentricularTachycardiaDuration | longest ventricular tachycardia duration | int ||
    | | longestVentricularTachycardiaOccurTime | longest ventricular tachycardia occurrence time | string ||
    | | longestVentricularTachycardiaCount | longest ventricular tachycardia count | int | signed report |
    | | maxVentricularPermatureHeartRate | ventricular tachycardia maximum heart rate | int | signature report |
    | supraventricularInfo | | | | supraventricular rhythm |
    | supraventricular | atrialBeatCount | total number of supraventricular beats | int ||
    | | atrialPermatureBeatCount | Number of supraventricular premature beats (single supraventricular premature beat) | int ||
    | | coupleAtrialPermatureCount | Paired supraventricular premature beats (paired supraventricular premature beats) | int ||
    | | atrialBigeminyCount | number of supraventricular bigeminys | int ||
    | | atrialTrigeminyCount | Number of supraventricular trigeminy | int ||
    | | atrialTachycardiaCount | Number of supraventricular tachycardia (number of episodes of supraventricular tachycardia) | int ||
    | | longestAtrialTachycardiaDuration | longest supraventricular tachycardia duration | int ||
    | | longestAtrialTachycardiaOccurTime | longest supraventricular tachycardia occurrence time | string||
    | | longestAtrialTachycardiaCount | longest supraventricular tachycardia count | int | signed report |
    | | maxAtrialPermatureHeartRate | Maximum supraventricular tachycardia heart rate | int | Signed report |
    | diagnose | | | | Diagnosis results|
    | conclusion | description | description conclusion | string ||
    | | diagnoseInfo | diagnostic conclusion | string ||
    | ecgHourEventList | | | | (signature report) hourly statistics list, single object attributes are as follows: |
    | hour event update table | hourName | time | string ||
    | | beatCount | beat count | int||
    | | minHeartRate | minimum | int ||
    | | averageHeartRate | average | int ||
    | | maxHeartRate | maximum | int ||
    | | asystoleCount | Stop betting | int ||
    | | ventricularBeatCount | ventricular beat | int ||
    | | coupleVentricularPermatureCount | Ventricular pairs | int ||
    | | ventricularTachycardiaCount | ventricular tachycardia | int ||
    | | atrialBeatCount | atrialBeatCount | int ||
    | | coupleAtrialPermatureCount | supraventricular sex pair | int ||
    | | atrialTachycardiaCount | supraventricular tachycardia | int ||
    | eventList | | | | Event fragment list, single object attributes are as follows: |
    | event list | eventCode | event encoding | string ||
    | | eventName | event description | string ||
    | | startPos | starting position | long ||
    | | endPos | end position | long ||
    | | eventTime | event time | string | will be affected by measure_time format |
    | | hr | heart rate | int ||
    | | leadNameList | The list of leads to print the display | List\| If empty, all leads will be printed |
    | hrvList | | (signed report) HRV hourly statistics list, single object attributes are as follows: |||
    | Heart rate variability list | tBeat | T.Beat | int ||
    | | qBeat | Q.Beat | int ||
    | | hr | HR | int ||
    | | rr | RR | int ||
    | | sdnn | SDNN | int ||
    | | sdann | SDANN | int ||
    | | sdnnIndex | SDNN* | int ||
    | | rmssd | rMSSD | int ||
    | | pnn50 | PNN50 | int ||
    | | ulf | ULF | int ||
    | | vlf | VLF | int ||
    | | lf | LF | int ||
    | | hf | HF | int ||
    | imgList | | | | (Signature report) Histogram, scatter plot data list |
    | Histogram | imgCode | Image encoding | string | "1-NN interval histogram, 2-NN interval difference histogram, 3-NN interval scatter plot, 4-RR interval scatter plot" |
    | | imgData | Image base64 encoded value | sting ||
    | imgSummary | | | | (Signature report) Histogram, scatter plot summary data |
    | Image description | maxRR | Max RR | double ||
    | | minRR | Min RR | double ||
    | | meanRR | Mean RR | double ||
    | | sdnn | SDNN | double ||
    | | triangularIndex | Triangular Index | double ||
    | | nn50 | NN50 | double ||
    | | pnn50 | PNN50 | double ||
    | | sdann | SDANN | double ||
    | | sdnnIndex | SDNN Index | double ||
    | | time | Time | string ||
    | | totalNumber | Total Number | int ||
    | | qualifiedNumber | Qualified Number | double ||
    | | qualified | Qualified | double ||
    | stList | | | | ST(mv) data list, the attributes of a single object are as follows: |
    | | leadName | lead name | string ||
    | | leadData | st data | List\
    ||
    | diagnoseList | | | List\
     | AI diagnosis content list (manual diagnosis content may be different from the above diagnostic results) |
    | | code | diagnostic code | string ||
    | | diagnoseInfo | diagnostic information | string ||

### Static ECG

1.ecgInfo

```jsonc
   {
    "ecg": {
        "duration": 370, //Analysis duration, unit s
        "sample_rate": 125, //Sampling rate, such as 125
        "measure_time": "2022-02-03 13:29:55" //Measurement time (YYYY-MM-DD HH:mm:ss format, such as 2022-02-04 13:29:55),
    },
    "user": {
        "name": "test1", //Name, required
        "phone": "", //Mobile phone number, required
        "gender": "1", //Gender (1: male, 2: female), required for signed reports
        "birthday": "1991-08-13", // Date of birth (yyyy-mm-dd), required for signature reports
        "id_number": "" //ID card number, required for signature reports
    },
    "device": {
        "sn": "236xxxx", //device sn
        "band": "Lepu", //Brand
        "model": "PC-700" //Model, find the corresponding correct model from the following device list (case sensitive)
    },
    "access_token": "", //AccessToken corresponding to service capabilities
    "analysis_type": "1-12", //Analysis type, before "-" indicates 1-short range, after "-" indicates lead type, 12-twelve leads
    "application_id": "xxx.xxx.xx", //Application id (application package name, pay attention to the format)
    "service_ability": 1 // 1 AI analysis, 2 doctor's signature report
   }
```

2. Equipment list

   | band | model | sampling rate | remarks |
   | ---- | ---------|-----| ---------------- |  
   | Lepu | PC-700 | 1000hz | PC700 all-in-one machine, 12 channels |
   | Lepu | PCECG-500| 1000hz | PCECG-500 all-in-one machine, 12 channels |


4. Description of analysis results

   | | |Chinese Description| Type|Remarks|
   |--------------------|---- |----------|-----|-----|
   | age | |age|string||
   | pid | | patient number | string ||
   | gender | |gender encoding|string||
   | genderText | | gender name |string ||
   | checkNo | | check number |string||
   | testTime | | Check the time | string | utc time, you need to add the local time zone to get the local time |
   | warnList | | Critical value list |object []||
   | itemCount | | |int||
   | patientId | | patient id |string||
   | diagnosedBy | | |string||
   | featureData | | Feature waveform data |object||
   | |scale |gain|float||
   | |filter |Filter parameters |int||
   | |sample |sampling rate |int||
   | |dataLen |data length|int||
   | |leadNum |Number of leads |int||
   | |testTime |Collection time|string||
   | |leadDatas |lead data|object []||
   | patientName | |Patient name|string||
   | measurements | | average measurement value |object||
   | featurePoints | |feature values ​​|object||
   | diagnosticList | |diagnostic list|object [] ||
   | testOfficeName | |Inspection Department|string||
   | diagnoseContent | |diagnosis result|string||
   | diagnoseBeginTime | | Diagnosis start time|string|utc time, you need to add the local time zone to get the local time|
   | diagnoseEndTime | | Diagnosis end time|string|utc time, you need to add the local time zone to get the local time|
   | diagnosedByName | | Name of diagnosed person |string||
   | leadMeasurements | | Measurements of each lead |object []||
   | diagnosedSignature | |diagnostic signature |string||
   | diagnoseFeatureDesc | |diagnosis feature description|string||

### Analysis status

 The 80002 state is an intermediate transition state, and the others are final states.  

   | code value | status |
   | ----- | ---------------- |
   | 80000 | ai analysis did not return status |
   | 80001 | Analysis completed |
   | 80002 | Analyzing |
   | 80003 | Analysis exception |
   | 80004 | Waiting for analysis file to be uploaded |
   | 80006 | Other exceptions |
