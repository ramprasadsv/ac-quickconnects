import groovy.json.JsonSlurper
import groovy.json.JsonOutput; 

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}

def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}

def checkList(qcName, tl) {
    boolean qcFound = false
    for(int i = 0; i < tl.QuickConnectSummaryList.size(); i++){
        def obj2 = tl.QuickConnectSummaryList[i]
        String qcName2 = obj2.Name
        if(qcName2.equals(qcName)) {
            qcFound = true
            break
        }
    }
    return qcFound
}

def getFlowId (primary, flowId, target) {
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = ""
    String rId = ""
    println "Searching for flowId : $flowId"
    for(int i = 0; i < pl.ContactFlowSummaryList.size(); i++){
        def obj = pl.ContactFlowSummaryList[i]    
        if (obj.Id.equals(flowId)) {
            fName = obj.Name
            println "Found flow name : $fName"
            break
        }
    }
    println "Searching for flow name : $fName"        
    for(int i = 0; i < tl.ContactFlowSummaryList.size(); i++){
        def obj = tl.ContactFlowSummaryList[i]    
        if (obj.Name.equals(fName)) {
            rId = obj.Id
            println "Found flow id : $rId"
            break
        }
    }
    return rId
}

def getQueueId (primary, queueId, target) {
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = ""
    String rId = ""
    println "Searching for queueId : $queueId"
    for(int i = 0; i < pl.QueueSummaryList.size(); i++){
        def obj = pl.QueueSummaryList[i]    
        if (obj.Id.equals(queueId)) {
            fName = obj.Name
            println "Found queue name : $fName"
            break
        }
    }
            
    for(int i = 0; i < tl.QueueSummaryList.size(); i++){
        def obj = tl.QueueSummaryList[i]    
        if (obj.Name.equals(fName)) {
            rId = obj.Id
            println "Found flow id : $rId"
            break
        }
    }
    return rId
    
}

def getUserId (primary, userId, target) {
    def pl = jsonParse(primary)
    def tl = jsonParse(target)
    String fName = ""
    String rId = ""
    println "Searching for userId : $userId"
    for(int i = 0; i < pl.UserSummaryList.size(); i++){
        def obj = pl.UserSummaryList[i]    
        if (obj.Id.equals(userId)) {
            fName = obj.Username
            println "Found user name : $fName"
            break
        }
    }
    println "Searching for userId for : $fName"        
    for(int i = 0; i < tl.UserSummaryList.size(); i++){
        def obj = tl.UserSummaryList[i]    
        if (obj.Username.equals(fName)) {
            rId = obj.Id
            println "Found flow id : $rId"
            break
        }
    }
    return rId
    
}


def INSTANCEARN = "662de594-7bab-4713-952b-2b4cb16f2724"
def FLOWID = "3b0db24a-c113-4847-8857-113c2c064131"
//def MISSINGQC = [:]
String MISSINGQC = ""
String TRAGETINSTANCEARN = "de1c040b-d1fe-4b12-b1e8-5e072329b86a"
String PRIMARYQC = ""
String TARGETQC = ""
String PRIMARYQUEUES = ""
String TARGETQUEUES = ""
String PRIMARYUSERS = ""
String TARGETUSERS = ""
String PRIMARYCFS = ""
String TARGETCFS = ""

pipeline {
    agent any
    stages {
        stage('git repo & clean') {
            steps {
                   sh(script: "rm -r ac-contactflows", returnStdout: true)
                   sh(script: "git clone https://github.com/ramprasadsv/ac-contactflows.git", returnStdout: true)
                   sh(script: "ls -ltr", returnStatus: true)
                   
            }
        }
        
        stage('List all Resources') {
            steps {
                echo "List all Resources in both instance "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        PRIMARYUSERS =  sh(script: "aws connect list-users --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYUSERS
                        TARGETUSERS =  sh(script: "aws connect  list-users --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETUSERS
                        
                        PRIMARYQUEUES =  sh(script: "aws connect list-queues --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYQUEUES
                        TARGETQUEUES =  sh(script: "aws connect list-queues --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETQUEUES
                        
                        PRIMARYQC =  sh(script: "aws connect list-quick-connects --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYQC
                        TARGETQC =  sh(script: "aws connect list-quick-connects --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETQC 
                        
                        PRIMARYCFS =  sh(script: "aws connect list-contact-flows --instance-id ${INSTANCEARN}", returnStdout: true).trim()
                        echo PRIMARYCFS
                        TARGETCFS =  sh(script: "aws connect list-contact-flows --instance-id ${TRAGETINSTANCEARN}", returnStdout: true).trim()
                        echo TARGETCFS
                    }
                }
            }
        }
        
        stage('List all quick connects') {
            steps {
                echo "List all quick in both instance "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def pl = jsonParse(PRIMARYQC)
                        def tl = jsonParse(TARGETQC)
                        int listSize = pl.QuickConnectSummaryList.size() 
                        println "Primary list size $listSize"
                        for(int i = 0; i < listSize; i++){
                            def obj = pl.QuickConnectSummaryList[i]
                            String qcName = obj.Name
                            String qcId = obj.Id
                            String qcType = obj.QuickConnectType
                            boolean qcFound = checkList(qcName, tl)
                            if(qcFound == false) {
                                println "Missing $qcName of type : $qcType -> $qcId"                                                              
                                MISSINGQC = MISSINGQC.concat(qcId).concat(",")                                
                            }
                        }
                        echo "Missing list -> ${MISSINGQC}"
                    }
                }
            }
        }
        
        stage('Find Missing quick connects') {
            steps {
                echo "Identify the quick connects "                
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {   
                    script {
                        def qcList = MISSINGQC.split(",")
                        for(int i = 0; i < qcList.size(); i++){
                            String qcId = qcList[i]
                            if(qcId.length() > 2){
                                def di =  sh(script: "aws connect describe-quick-connect --instance-id ${INSTANCEARN} --quick-connect-id ${qcId}", returnStdout: true).trim()
                                echo di
                                def qc = jsonParse(di)
                                if(qc.QuickConnect.QuickConnectConfig.QuickConnectType.equals("PHONE_NUMBER")){
                                    String qcName = qc.QuickConnect.Name
                                    String qcDesc = qc.QuickConnect.Description
                                    String qcPhoneNumber = qc.QuickConnect.QuickConnectConfig.PhoneConfig.PhoneNumber
                                    String qcConfig = "QuickConnectType=PHONE_NUMBER,PhoneConfig={PhoneNumber=${qcPhoneNumber}}"
                                    qc = null
                                    def dq =  sh(script: "aws connect create-quick-connect --instance-id ${TRAGETINSTANCEARN} --name ${qcName} --description ${qcDesc} --quick-connect-config ${qcConfig}", returnStdout: true).trim()
                                    echo dq
                                    
                                }else if(qc.QuickConnect.QuickConnectConfig.QuickConnectType.equals("USER")){
                                    String userId = qc.QuickConnect.QuickConnectConfig.UserConfig.UserId 
                                    String flowId = qc.QuickConnect.QuickConnectConfig.UserConfig.ContactFlowId
                                    String qcName = qc.QuickConnect.Name
                                    String qcDesc = qc.QuickConnect.Description
                                    qc = null
                                    String targetFlowId = getFlowId (PRIMARYCFS, flowId, TARGETCFS)
                                    String targetUserId = getUserId (PRIMARYUSERS, userId, TARGETUSERS)
                                    String qcConfig = "QuickConnectType=USER,UserConfig=\\{UserId=" + targetUserId +",ContactFlowId=" + targetFlowId + "\\}"
                                    def cu =  sh(script: "aws connect create-quick-connect --instance-id ${TRAGETINSTANCEARN} --name ${qcName} --description ${qcDesc} --quick-connect-config ${qcConfig}", returnStdout: true).trim()
                                    echo cu
                                }else{                                    
                                    String queueId = qc.QuickConnect.QuickConnectConfig.QueueConfig.QueueId
                                    String flowId = qc.QuickConnect.QuickConnectConfig.QueueConfig.ContactFlowId
                                    String qcName = qc.QuickConnect.Name
                                    String qcDesc = qc.QuickConnect.Description
                                    qc = null
                                    String targetFlowId = getFlowId (PRIMARYCFS, flowId, TARGETCFS)
                                    String targetQueueId = getQueueId (PRIMARYQUEUES, queueId, TARGETQUEUES)
                                    String qcConfig = "QuickConnectType=QUEUE,QueueConfig=\\{QueueId=" + targetQueueId + ",ContactFlowId=" + targetFlowId +"\\}"                                    
                                    def cq =  sh(script: "aws connect create-quick-connect --instance-id ${TRAGETINSTANCEARN} --name ${qcName} --description ${qcDesc} --quick-connect-config ${qcConfig}" , returnStdout: true).trim()
                                    echo cq
                                }
                            }
                        }
                    }                
                }
            }
        } 

         stage('Create Missing quick connects') {
            steps {
                echo "Create the quick connects that were missing"                
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {   
                }
            } 
         }
        
     }
}
