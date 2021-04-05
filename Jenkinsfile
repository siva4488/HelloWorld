pipeline 
{
    agent any
    environment 
	{
        /********** GLOBAL environment variables applicable to all HCMX offerings ************/
		// HCMX Server's fully qualified domain name
		HCMX_SERVER_FQDN = "catvmlmpoc1.ftc.hpeswlab.net"
        
		// HCMX tenant's ID that has DND capability. DND capability is required to provision and manage VMs.
		// HCMX will be used to provision VMs on which testing of the new build will be performed.
		// After testing is complete, provisioned VMs are deleted so that expenses on public cloud is reduced and resource usage on private cloud is reduced.
		HCMX_TENANT_ID = "616409711"

		// Define interval in seconds to check status of VM deployment request in HCMX
		// VM deployment may take longer than 10 minutes depending on cloud provider.
		HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS = "30"
		
		// Set this to zero seconds if you are using it in productions jenkins environment.
		// Set this to atleast 180 seconds for demonstration of deployed VM using HCMX
		HCMX_SUB_CANCEL_DELAY_SECONDS = "0"
		
		// If test VM is not provisioned by HCMX within the time specified in this parameter, exit the build.
		HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS = "600"
		
		// If Jenkins needs a web proxy to reach HCMX, set this to YES. Possible values are YES and NO.
		USE_PROXY = "YES"
		
		// Web proxy hostname. This is an optional parameter when USE_PROXY is set to NO. If USE_PROXY is set to YES, then PROXY_HOST is mandatory.
		//PROXY_HOST = "web-proxy.us.softwaregrp.net"
		PROXY_HOST = ""
		
		// Web proxy port. This is an optional parameter when USE_PROXY is set to NO. If USE_PROXY is set to YES, then PROXY_PORT is mandatory.
		PROXY_PORT = "8080"
		
		// Web proxy protocol. This is an optional parameter when USE_PROXY is set to NO. If USE_PROXY is set to YES, then PROXY_PROTOCOL is mandatory.
		PROXY_PROTOCOL = "http"
		
		// Web proxy credentials. This is an optional parameter when USE_PROXY is set to NO. If USE_PROXY is set to YES, then PROXY_REQUIRES_CREDENTIALS is mandatory.
		// Create user name, password credentials in Jenkins with ProxyCred as its ID.
		PROXY_REQUIRES_CREDENTIALS = "YES"
		
		
		/********** HCMX Offering specific environment variables. In this example, this section is for HCMX offering to deploy VMs on vCenter ************/
		// VMWare vCenter data center in which VM has to be deployed
		HCMX_VCENTER_DATACENTER = "CAT"
		
		//VMWare vCenter template to be used for deployment of VM
		HCMX_VCENTER_VM_TEMPLATE = "catvmlmdep_t"
		
		//VMWare vCenter custom spec to be used for deployment of VM
		HCMX_VCENTER_VM_CUSTOMSPEC = "(Ts)catvmLinuxDHCP"
		
		//VM name prefix to be used during deployment of VM
		HCMX_VCENTER_VMNAME_PREFIX = "HelloWorld"
		
		// Memory size in MB to be used for the deployment of VM
		HCMX_VCENTER_VM_MEMORY_SIZE_MB = "1024"
		
		// Number of CPUs to be used for the deployment of VM
		HCMX_VCENTER_VM_NUM_CPU = "1"

		// Number of Virtual Machines needed to conduct testing
		HCMX_VCENTER_VM_NUM = "1"
		
		// Request title displayed in HCMX
		HCMX_VCENTER_VM_REQUEST_TITLE = "Request to deploy a new VM to test Hello World App"
		
		// Request description to deploy a new VM for testing the build
		HCMX_VCENTER_VM_REQUEST_DESCRIPTION = "Need VM to test Hello World Application"
		
		// HCMX subscription name. Each service deployed through HCMX has a subscription associated with it.
		HCMX_VCENTER_VM_SUB_NAME = "Hello World Test VM"
		
		// HCMX subscription description
		HCMX_VCENTER_VM_SUB_DESCRIPTION = "Hello World Test VM"		
		
    }

    stages 
	{
        stage('Build') 
		{
            steps 
			{
                /*  Build your project in this section.
					You may use maven, ant, etc to build your project.
					Compile code. Perform unit test and integration tests.
					In this example use case, a sample HelloWorld.sh script that prints "Hello World" is built.
					Neither Maven nor ant is required. The HelloWorld.sh is used directly as from source as the compiled code in this example use case.
				*/
				
				echo '***************************************** BUILDING *****************************************'
				sh 'mkdir build'
				sh 'cp HelloWorld.sh build/HelloWorld.sh'
				sh 'chmod 555 build/HelloWorld.sh'
				sh "echo version = 1.0.${env.BUILD_ID} >> build/version.txt"				
            }
        }
        stage('Test') 
		{
            steps 
			{
                /*  Test your project in this section.
					Perform UAT testing on the test server.
					In this use case example, a new VM is provisioned using HCMX. 
					Build is transferred to the new VM. Testing of build is conducted on the new VM.
					After testing is complete, the new VM is deleted through HCMX to release resource usage on the cloud provider.
				*/
				
				echo '***************************************** PROVISIONING VM(s) THROUGH HCMX FOR TESTING THE BUILD  *****************************************'				
				
				script 
				{
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'HCMXUser', usernameVariable: 'HCMX_USER', passwordVariable: 'HCMX_USER_PSW']]) 
					{
						withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'ProxyCred', usernameVariable: 'PROXY_USER', passwordVariable: 'PROXY_USER_PSW']]) 
						{	    
							final int PROXY_PORT_NUM_MIN = 0
							final int PROXY_PORT_NUM_MAX = 65535
							
							// Minimum memory size in MB that must be specified to deploy VMs
							final int HCMX_VCENTER_VM_MIN_MEMORY_SIZE = 4
			
							// Maximum memory size in MB that can be specified to deploy VMs
							final int HCMX_VCENTER_VM_MAX_MEMORY_SIZE = 6275072
							
							// Minimum number of CPUs that must be specified to deploy VM
							final int HCMX_VCENTER_VM_MIN_NUM_CPU = 1
														
							// Maximum number of CPUs that can be specified to deploy VM
							final int HCMX_VCENTER_VM_MAX_NUM_CPU = 32	
							
							// Minimum number of VMs that must be specified to deploy VM
							final int HCMX_VCENTER_VM_MIN_NUM = 1
							
							String HCMX_SERVER_FQDN
							String HCMX_TENANT_ID
							int HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS = 30
							int HCMX_SUB_CANCEL_DELAY_SECONDS = 0						
							int HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS = 600
							String USE_PROXY
							String PROXY_HOST
							int PROXY_PORT
							String PROXY_PROTOCOL
							String PROXY_REQUIRES_CREDENTIALS
													
							String HCMX_VCENTER_DATACENTER
							String HCMX_VCENTER_VM_TEMPLATE
							String HCMX_VCENTER_VM_CUSTOMSPEC
							String HCMX_VCENTER_VMNAME_PREFIX						
							long HCMX_VCENTER_VM_MEMORY_SIZE_MB = 1024
							int HCMX_VCENTER_VM_NUM_CPU = 1
							int HCMX_VCENTER_VM_NUM = 1
							String HCMX_VCENTER_VM_REQUEST_TITLE
							String HCMX_VCENTER_VM_REQUEST_DESCRIPTION
							String HCMX_VCENTER_VM_SUB_NAME
							String HCMX_VCENTER_VM_SUB_DESCRIPTION
							
							String curlCMD = "curl"
							
							if(env.HCMX_SERVER_FQDN)
							{
								HCMX_SERVER_FQDN = env.HCMX_SERVER_FQDN
							}
							else
							{
								error "HCMX_SERVER_FQDN cannot be NULL or empty"
							}
							
							if(env.HCMX_TENANT_ID)
							{
								HCMX_TENANT_ID = env.HCMX_TENANT_ID
							}
							else
							{
								error "HCMX_TENANT_ID cannot be NULL or empty"
							}
																			
							if (env.HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS && env.HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS.toString().isNumber())
							{
								HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS = env.HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS as int
							}
							else
							{
								error "HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS must be an integer"
							}
							
							if (env.HCMX_SUB_CANCEL_DELAY_SECONDS && env.HCMX_SUB_CANCEL_DELAY_SECONDS.toString().isNumber())
							{
								HCMX_SUB_CANCEL_DELAY_SECONDS = env.HCMX_SUB_CANCEL_DELAY_SECONDS as int
							}
							else
							{
								error "HCMX_SUB_CANCEL_DELAY_SECONDS must be an integer"
							}
							
							if (env.HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS && env.HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS.toString().isNumber())
							{
								HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS = env.HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS as int
							}
							else
							{
								error "HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS must be an integer"
							}
							
							if(env.USE_PROXY)
							{
								USE_PROXY = env.USE_PROXY
								if (USE_PROXY && (!(USE_PROXY.equalsIgnoreCase("YES")) && !(USE_PROXY.equalsIgnoreCase("NO"))))
								{
									error "USE_PROXY must be set to either YES or NO"
								}
							}
							else
							{
								error "USE_PROXY cannot be NULL or empty"
							}
							
							if (USE_PROXY && USE_PROXY.equalsIgnoreCase("YES"))
							{
								if(env.PROXY_HOST)
								{
									PROXY_HOST = env.PROXY_HOST
								}
								else
								{
									error "PROXY_HOST cannot be NULL or empty when USE_PROXY is set to yes"
								}
							
								if(env.PROXY_PORT && env.PROXY_PORT.toString().isNumber())
								{
									PROXY_PORT = env.PROXY_PORT as int
									if (PROXY_PORT < PROXY_PORT_NUM_MIN || PROXY_PORT > PROXY_PORT_NUM_MAX)
									{
										error "PROXY_PORT must be >= $PROXY_PORT_NUM_MIN and <= $PROXY_PORT_NUM_MAX"
									}
								}
								else
								{
									error "PROXY_PORT cannot be NULL or empty when USE_PROXY is set to yes"
								}
							
								if(env.PROXY_PROTOCOL)
								{
									PROXY_PROTOCOL = env.PROXY_PROTOCOL
									if (PROXY_PROTOCOL && (!(PROXY_PROTOCOL.equalsIgnoreCase("HTTP")) && !(PROXY_PROTOCOL.equalsIgnoreCase("HTTPS"))))
									{
										error "PROXY_PROTOCOL must be set to either http or https"
									}
								}
								else
								{
									error "PROXY_PROTOCOL cannot be NULL or empty when USE_PROXY is set to yes"
								}						
								if(env.PROXY_REQUIRES_CREDENTIALS)
								{
									PROXY_REQUIRES_CREDENTIALS = env.PROXY_REQUIRES_CREDENTIALS
									if (PROXY_REQUIRES_CREDENTIALS && (!(PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("YES")) && !(PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
									{
										error "PROXY_REQUIRES_CREDENTIALS must be set to either YES or NO"
									}
								}
								else
								{
									error "PROXY_REQUIRES_CREDENTIALS cannot be NULL or empty when USE_PROXY is set to yes"
								}
								curlCMD = "curl --proxy \""+ PROXY_PROTOCOL + "://" + PROXY_HOST + ":" + PROXY_PORT +"\""								
							}
							else
							{
								curlCMD = "curl"
							}
							
							if(env.HCMX_VCENTER_DATACENTER)
							{
								HCMX_VCENTER_DATACENTER = env.HCMX_VCENTER_DATACENTER
							}
							else
							{
								error "HCMX_VCENTER_DATACENTER cannot be NULL or empty"
							}											
							
							if(env.HCMX_VCENTER_VM_TEMPLATE)
							{
								HCMX_VCENTER_VM_TEMPLATE = env.HCMX_VCENTER_VM_TEMPLATE
							}
							else
							{
								error "HCMX_VCENTER_VM_TEMPLATE cannot be NULL or empty"
							}
							
							if(env.HCMX_VCENTER_VM_CUSTOMSPEC)
							{
								HCMX_VCENTER_VM_CUSTOMSPEC = env.HCMX_VCENTER_VM_CUSTOMSPEC
							}
							else
							{
								error "HCMX_VCENTER_VM_CUSTOMSPEC cannot be NULL or empty"
							}
							
							if(env.HCMX_VCENTER_VMNAME_PREFIX)
							{
								HCMX_VCENTER_VMNAME_PREFIX = env.HCMX_VCENTER_VMNAME_PREFIX
							}
							else
							{
								error "HCMX_VCENTER_VMNAME_PREFIX cannot be NULL or empty"
							}
							
							if (env.HCMX_VCENTER_VM_MEMORY_SIZE_MB && env.HCMX_VCENTER_VM_MEMORY_SIZE_MB.toString().isNumber())
							{
								HCMX_VCENTER_VM_MEMORY_SIZE_MB = env.HCMX_VCENTER_VM_MEMORY_SIZE_MB as long
								if (HCMX_VCENTER_VM_MEMORY_SIZE_MB < HCMX_VCENTER_VM_MIN_MEMORY_SIZE || HCMX_VCENTER_VM_MEMORY_SIZE_MB > HCMX_VCENTER_VM_MAX_MEMORY_SIZE )
								{
									error "HCMX_VCENTER_VM_MEMORY_SIZE_MB in MB must be >= $HCMX_VCENTER_VM_MIN_MEMORY_SIZE and HCMX_VCENTER_VM_MEMORY_SIZE_MB in MB must be <= $HCMX_VCENTER_VM_MAX_MEMORY_SIZE"
								}							
							}
							else
							{
								error "HCMX_VCENTER_VM_MEMORY_SIZE_MB must be an integer"
							}
							
							if (env.HCMX_VCENTER_VM_NUM_CPU && env.HCMX_VCENTER_VM_NUM_CPU.toString().isNumber())
							{
								HCMX_VCENTER_VM_NUM_CPU = env.HCMX_VCENTER_VM_NUM_CPU as int
								if (HCMX_VCENTER_VM_NUM_CPU < HCMX_VCENTER_VM_MIN_NUM_CPU || HCMX_VCENTER_VM_NUM_CPU > HCMX_VCENTER_VM_MAX_NUM_CPU )
								{
									error "HCMX_VCENTER_VM_NUM_CPU must be >= $HCMX_VCENTER_VM_MIN_NUM_CPU and HCMX_VCENTER_VM_NUM_CPU must be <= $HCMX_VCENTER_VM_MAX_NUM_CPU"
								}		
							}
							else
							{
								error "HCMX_VCENTER_VM_NUM_CPU must be an integer"
							}
							
							if (env.HCMX_VCENTER_VM_NUM && env.HCMX_VCENTER_VM_NUM.toString().isNumber())
							{
								HCMX_VCENTER_VM_NUM = env.HCMX_VCENTER_VM_NUM as int
								if (HCMX_VCENTER_VM_NUM < HCMX_VCENTER_VM_MIN_NUM)
								{
									error "HCMX_VCENTER_VM_NUM must be >= $HCMX_VCENTER_VM_MIN_NUM"
								}		
							}
							else
							{
								error "HCMX_VCENTER_VM_NUM must be an integer"
							}
							
							if(env.HCMX_VCENTER_VM_REQUEST_TITLE)
							{
								HCMX_VCENTER_VM_REQUEST_TITLE = env.HCMX_VCENTER_VM_REQUEST_TITLE
							}
							else
							{
								error "HCMX_VCENTER_VM_REQUEST_TITLE cannot be NULL or empty"
							}
							
							if(env.HCMX_VCENTER_VM_REQUEST_DESCRIPTION)
							{
								HCMX_VCENTER_VM_REQUEST_DESCRIPTION = env.HCMX_VCENTER_VM_REQUEST_DESCRIPTION
							}
							else
							{
								error "HCMX_VCENTER_VM_REQUEST_DESCRIPTION cannot be NULL or empty"
							}
							
							if(env.HCMX_VCENTER_VM_SUB_NAME)
							{
								HCMX_VCENTER_VM_SUB_NAME = env.HCMX_VCENTER_VM_SUB_NAME
							}
							else
							{
								error "HCMX_VCENTER_VM_SUB_NAME cannot be NULL or empty"
							}
							
							if(env.HCMX_VCENTER_VM_SUB_DESCRIPTION)
							{
								HCMX_VCENTER_VM_SUB_DESCRIPTION = env.HCMX_VCENTER_VM_SUB_DESCRIPTION
							}
							else
							{
								error "HCMX_VCENTER_VM_SUB_DESCRIPTION cannot be NULL or empty"
							}				
							
							if (HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS >= HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS)
							{
								error "HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS must be less than HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS."
							}
							
							echo "HCMX: Get SMAX Auth Token"
							// HCMX REST APIs require SMAX AUTH TOKEN and TENANT ID to perform any POST, PUT and GET operations.
							// Build HCMX Authentication Token URL
							final String HCMX_AUTH_URL = "https://" + HCMX_SERVER_FQDN + "/auth/authentication-endpoint/authenticate/token?TENANTID=" + HCMX_TENANT_ID
							String SMAX_AUTH_TOKEN
							int getTokenResCode
							
							// Submit a REST API call to HCMX to get SMAX_AUTH_TOKEN
							if (USE_PROXY.equalsIgnoreCase("NO") || ((USE_PROXY.equalsIgnoreCase("YES")) && (PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
							{
								(SMAX_AUTH_TOKEN, getTokenResCode) = sh(script: '''set +x;''' + curlCMD + ''' -s -w \'\\n%{response_code}\' -X POST ''' + HCMX_AUTH_URL + ''' -k -H "Content-Type: application/json" -d \'{"login":"\'"$HCMX_USER"\'","password":"\'"$HCMX_USER_PSW"\'"}\' ''', returnStdout: true).trim().tokenize("\n")
							}
							else
							{
								(SMAX_AUTH_TOKEN, getTokenResCode) = sh(script: '''set +x;''' + curlCMD + ''' --proxy-user $PROXY_USER:$PROXY_USER_PSW   -s -w \'\\n%{response_code}\' -X POST ''' + HCMX_AUTH_URL + ''' -k -H "Content-Type: application/json" -d \'{"login":"\'"$HCMX_USER"\'","password":"\'"$HCMX_USER_PSW"\'"}\' ''', returnStdout: true).trim().tokenize("\n")
							}
							
							if (getTokenResCode == 200 && SMAX_AUTH_TOKEN && SMAX_AUTH_TOKEN.trim())
							{											
								echo "HCMX: Get person ID"
								// Build HCMX Get Person ID URL
								//final String HCMX_GET_PERSON_ID_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ems/Person?filter=(Upn=%27" + HCMX_USER + "%27)&layout=Id"
								final String HCMX_GET_PERSON_ID_URL_1 = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ems/Person?filter=(Upn=%27"
								final String HCMX_GET_PERSON_ID_URL_2 = "%27)&layout=Id"
								String personIDResponse
								int personIDResCode
								
								// Submit a REST API call to HCMX to get Person ID
								if (USE_PROXY.equalsIgnoreCase("NO") || ((USE_PROXY.equalsIgnoreCase("YES")) && (PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
								{
									(personIDResponse, personIDResCode)  = sh(script: '''set +x;''' + curlCMD + ''' -s -w '\\n%{response_code}' "''' + HCMX_GET_PERSON_ID_URL_1 + HCMX_USER + HCMX_GET_PERSON_ID_URL_2 + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
								}
								else
								{
									(personIDResponse, personIDResCode)  = sh(script: '''set +x;''' + curlCMD + ''' --proxy-user $PROXY_USER:$PROXY_USER_PSW -s -w '\\n%{response_code}' "''' + HCMX_GET_PERSON_ID_URL_1 + HCMX_USER + HCMX_GET_PERSON_ID_URL_2 + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
								}
								
								if (personIDResCode == 200 && personIDResponse && personIDResponse.trim()) 
								{
									def personIDResponseJSON = new groovy.json.JsonSlurperClassic().parseText(personIDResponse)							
									def HCMX_PERSON_ID = personIDResponseJSON.entities[0].properties.Id
									echo "HCMX Requested for person ID is $HCMX_PERSON_ID"
								
									echo "HCMX: Submit request for a new VM for testing"
									// Build HCMX create request URL
									final String HCMX_CREATE_REQUEST_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ess/request/createRequest"
									
									Date curDate = new Date()
									long epochMilliSeconds = curDate.getTime()
									String depVMResponse
									int depVMResponseCode
									
									// Submit a REST API call to HCMX to deploy a new test server VM								
									if (USE_PROXY.equalsIgnoreCase("NO") || ((USE_PROXY.equalsIgnoreCase("YES")) && (PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
									{
										(depVMResponse, depVMResponseCode) = sh(script: '''set +x;''' + curlCMD + ''' -s -w '\\n%{response_code}' -X POST "''' + HCMX_CREATE_REQUEST_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" -d '{"entities":[{"entity_type":"Request","properties":{"RequestedForPerson":"''' + HCMX_PERSON_ID + '''","StartDate":''' + epochMilliSeconds + ''',"RequestsOffering":"10096","CreationSource":"CreationSourceEss","RequestedByPerson":"''' + HCMX_PERSON_ID + '''","DataDomains":["Public"],"CreateTime":''' + epochMilliSeconds + ''',"UserOptions":"{\\"complexTypeProperties\\":[{\\"properties\\":{\\"OptionSet0c6eb101a1a178c3c49c3badbc481f05_c\\":{\\"Option34c8d8d8403ac43361b8b8083004ef4a_c\\":true},\\"OptionSet2ee4a8f73fcd1606c1337172e8411e2a_c\\":{\\"Option19cd6cd22067142e0977622ed71ced7d_c\\":true},\\"OptionSet473C6F2BE6F45DB8381664FC9097BE37_c\\":{\\"Option2E8493EA9AC2821929DA64FC90978A98_c\\":true},\\"changedUserOptionsForSimulation\\":\\"PropertyvmCpuCount19cd6cd22067142e0977622ed71ced7d_c&\\",\\"PropertyserverCount34c8d8d8403ac43361b8b8083004ef4a_c\\":\\"''' + HCMX_VCENTER_VM_NUM + '''\\",\\"PropertyproviderId2E8493EA9AC2821929DA64FC90978A98_c\\":\\"2c908fac77eefca5017822299d726af6\\",\\"PropertydatacenterName2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_DATACENTER + '''\\",\\"PropertyvirtualMachine2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_VM_TEMPLATE + '''\\",\\"PropertycustomizationTemplateName2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_VM_CUSTOMSPEC + '''\\",\\"PropertyvmNamePrefix2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_VMNAME_PREFIX + '''\\",\\"Option19cd6cd22067142e0977622ed71ced7d_c\\":true,\\"Optionad52a8efe1465faa8c389ae92bf90d0c_c\\":false,\\"PropertyvmMemorySize19cd6cd22067142e0977622ed71ced7d_c\\":\\"''' + HCMX_VCENTER_VM_MEMORY_SIZE_MB + '''\\",\\"PropertyvmCpuCount19cd6cd22067142e0977622ed71ced7d_c\\":\\"''' + HCMX_VCENTER_VM_NUM_CPU + '''\\"}}]}","Description":"<p>''' + HCMX_VCENTER_VM_REQUEST_DESCRIPTION + '''</p>","RelatedSubscriptionName":"''' + HCMX_VCENTER_VM_SUB_NAME + '''","RelatedSubscriptionDescription":"<p>''' + HCMX_VCENTER_VM_SUB_DESCRIPTION + '''</p>","RequestAttachments":"{\\"complexTypeProperties\\":[]}","DisplayLabel":"''' + HCMX_VCENTER_VM_REQUEST_TITLE + '''"}}],"operation":"CREATE"}' ''', returnStdout: true).trim().tokenize("\n")	
									}
									else
									{
										(depVMResponse, depVMResponseCode) = sh(script: '''set +x;''' + curlCMD + ''' --proxy-user $PROXY_USER:$PROXY_USER_PSW -s -w '\\n%{response_code}' -X POST "''' + HCMX_CREATE_REQUEST_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" -d '{"entities":[{"entity_type":"Request","properties":{"RequestedForPerson":"''' + HCMX_PERSON_ID + '''","StartDate":''' + epochMilliSeconds + ''',"RequestsOffering":"10096","CreationSource":"CreationSourceEss","RequestedByPerson":"''' + HCMX_PERSON_ID + '''","DataDomains":["Public"],"CreateTime":''' + epochMilliSeconds + ''',"UserOptions":"{\\"complexTypeProperties\\":[{\\"properties\\":{\\"OptionSet0c6eb101a1a178c3c49c3badbc481f05_c\\":{\\"Option34c8d8d8403ac43361b8b8083004ef4a_c\\":true},\\"OptionSet2ee4a8f73fcd1606c1337172e8411e2a_c\\":{\\"Option19cd6cd22067142e0977622ed71ced7d_c\\":true},\\"OptionSet473C6F2BE6F45DB8381664FC9097BE37_c\\":{\\"Option2E8493EA9AC2821929DA64FC90978A98_c\\":true},\\"changedUserOptionsForSimulation\\":\\"PropertyvmCpuCount19cd6cd22067142e0977622ed71ced7d_c&\\",\\"PropertyserverCount34c8d8d8403ac43361b8b8083004ef4a_c\\":\\"''' + HCMX_VCENTER_VM_NUM + '''\\",\\"PropertyproviderId2E8493EA9AC2821929DA64FC90978A98_c\\":\\"2c908fac77eefca5017822299d726af6\\",\\"PropertydatacenterName2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_DATACENTER + '''\\",\\"PropertyvirtualMachine2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_VM_TEMPLATE + '''\\",\\"PropertycustomizationTemplateName2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_VM_CUSTOMSPEC + '''\\",\\"PropertyvmNamePrefix2E8493EA9AC2821929DA64FC90978A98_c\\":\\"''' + HCMX_VCENTER_VMNAME_PREFIX + '''\\",\\"Option19cd6cd22067142e0977622ed71ced7d_c\\":true,\\"Optionad52a8efe1465faa8c389ae92bf90d0c_c\\":false,\\"PropertyvmMemorySize19cd6cd22067142e0977622ed71ced7d_c\\":\\"''' + HCMX_VCENTER_VM_MEMORY_SIZE_MB + '''\\",\\"PropertyvmCpuCount19cd6cd22067142e0977622ed71ced7d_c\\":\\"''' + HCMX_VCENTER_VM_NUM_CPU + '''\\"}}]}","Description":"<p>''' + HCMX_VCENTER_VM_REQUEST_DESCRIPTION + '''</p>","RelatedSubscriptionName":"''' + HCMX_VCENTER_VM_SUB_NAME + '''","RelatedSubscriptionDescription":"<p>''' + HCMX_VCENTER_VM_SUB_DESCRIPTION + '''</p>","RequestAttachments":"{\\"complexTypeProperties\\":[]}","DisplayLabel":"''' + HCMX_VCENTER_VM_REQUEST_TITLE + '''"}}],"operation":"CREATE"}' ''', returnStdout: true).trim().tokenize("\n")																		
									}
																	
													
									if (depVMResponseCode == 200 && depVMResponse && depVMResponse.trim()) 
									{
										def depVMResponseJSON = new groovy.json.JsonSlurperClassic().parseText(depVMResponse)								
										def HCMX_REQUEST_ID = depVMResponseJSON.entity_result_list.entity[0].properties.Id
										echo "HCMX Request ID to deploy a new test server VM is $HCMX_REQUEST_ID"
										
										// Build HCMX Get request status URL
										final String HCMX_GET_REQUEST_STATUS_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ems/Request?filter=(Id=%27" + HCMX_REQUEST_ID + "%27)&layout=PhaseId"
										String reqStatus = "Log"
										int reqCode = 0
										int timeSpent = 0
										String reqResponse = ""
										
										// Loop until Request status changes to Close. Once it is in closed status VM is deployed and ready for testing.
										while (reqStatus != 'Close')
										{
											if (timeSpent > HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS)
											{
												error "Failed to provision VM deployment within the timeout period of $HCMX_REQ_DEPLOY_TESTVM_TIMEOUT_SECONDS seconds"										
											}
											// Submit a REST API call to HCMX to get status of VM deployment request
											echo "HCMX: Get request status until it is Closed"
											
											if (USE_PROXY.equalsIgnoreCase("NO") || ((USE_PROXY.equalsIgnoreCase("YES")) && (PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
											{
												(reqResponse, reqCode) = sh(script: '''set +x;''' + curlCMD + ''' -s -w '\\n%{response_code}' "''' + HCMX_GET_REQUEST_STATUS_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
											}
											else
											{
												(reqResponse, reqCode) = sh(script: '''set +x;''' + curlCMD + ''' --proxy-user $PROXY_USER:$PROXY_USER_PSW -s -w '\\n%{response_code}' "''' + HCMX_GET_REQUEST_STATUS_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
											}
											
											if (reqCode == 200 && reqResponse && reqResponse.trim()) 
											{
												def reqResponseJSON = new groovy.json.JsonSlurperClassic().parseText(reqResponse)
												reqStatus = reqResponseJSON.entities[0].properties.PhaseId
												echo "HCMX REQUEST status is $reqStatus"
												if (reqStatus && reqStatus.equalsIgnoreCase("Close"))
												{
													// If request for VM deployment has moved to Close phase, then VM has been deployed successfully.
													break;
												}
												else
												{
													echo "sleep for $HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS seconds before checking request status again"
													sleep(HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS)
													timeSpent = timeSpent + HCMX_REQ_STATUS_CHK_INTERVAL_SECONDS
												}                                        
											}
											else
											{
												error "Failed to get request status for VM deployment from HCMX"
											}
										}
										
										echo "HCMX: Get subscription ID"
										// Build HCMX Get subscription URL using the request ID that was obtained in earlier steps.
										final String HCMX_GET_SUBSCRIPTION_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ems/Subscription?filter=(InitiatedByRequest=%27" + HCMX_REQUEST_ID + "%27%20and%20Status=%27Active%27)&layout=Id"
										
										String subResponse
										int subResCode
										// Submit a REST API call to HCMX to get subscription ID
										if (USE_PROXY.equalsIgnoreCase("NO") || ((USE_PROXY.equalsIgnoreCase("YES")) && (PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
										{
											(subResponse, subResCode)  = sh(script: '''set +x;''' + curlCMD + ''' -s -w '\\n%{response_code}' "''' + HCMX_GET_SUBSCRIPTION_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
										}
										else
										{
											(subResponse, subResCode)  = sh(script: '''set +x;''' + curlCMD + ''' --proxy-user $PROXY_USER:$PROXY_USER_PSW -s -w '\\n%{response_code}' "''' + HCMX_GET_SUBSCRIPTION_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
										}
										
										if (subResCode == 200 && subResponse && subResponse.trim()) 
										{
											def subResponseJSON = new groovy.json.JsonSlurperClassic().parseText(subResponse)									
											subID = subResponseJSON.entities[0].properties.Id
											echo "HCMX Subscription ID is $subID" 
											
											echo "HCMX: Get service instances"
											// Prepare HCMX Get service instance URL using the subscription ID that was obtained in earlier steps.
											final String HCMX_GET_SVCINSTANCE_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/cloud-service/getServiceInstance/" + subID
											
											String svcInstResponse
											int svcInstResCode
											// Submit a REST API call to HCMX to get a list of service instances associated with the subscription
											if (USE_PROXY.equalsIgnoreCase("NO") || ((USE_PROXY.equalsIgnoreCase("YES")) && (PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
											{	
												(svcInstResponse, svcInstResCode)  = sh(script: '''set +x;''' + curlCMD + ''' -s -w '\\n%{response_code}' "''' + HCMX_GET_SVCINSTANCE_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
											}
											else
											{
												(svcInstResponse, svcInstResCode)  = sh(script: '''set +x;''' + curlCMD + ''' --proxy-user $PROXY_USER:$PROXY_USER_PSW -s -w '\\n%{response_code}' "''' + HCMX_GET_SVCINSTANCE_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")											
											}
											
											if (svcInstResCode == 200 && svcInstResponse && svcInstResponse.trim()) 
											{
												def svcInstResponseJSON = new groovy.json.JsonSlurperClassic().parseText(svcInstResponse)
												def svcInstTopologyArray = svcInstResponseJSON.topology												
												List<String> testVMIPList = new ArrayList<String>();
												
												echo "HCMX: Looping through service instances to find IP address of deployed VM"
												// Loop through service instances. Retrieve IP address property value for the service instance with type = CI_TYPE_SERVER
												for(def member : svcInstTopologyArray) 
												{
													if(member.type.name && member.type.name.equalsIgnoreCase("CI_TYPE_SERVER")) 
													{
														def svcInstPropertyArray = member.properties
														for(def propMember : svcInstPropertyArray) 
														{
															if(propMember.name && propMember.name.equalsIgnoreCase("primary_ip_address"))
															{
																testVMIPList.add(propMember.propertyValue)																										 
																break
															}
														}														
													}
												}
												if (testVMIPList.size() == 0)
												{
													echo "Deployed VM's IP address is empty. Cannot copy build to test on newly deployed VMs"
													CancelSubscription(HCMX_SUB_CANCEL_DELAY_SECONDS, HCMX_SERVER_FQDN, HCMX_TENANT_ID, HCMX_PERSON_ID, subID, SMAX_AUTH_TOKEN, curlCMD, USE_PROXY, PROXY_REQUIRES_CREDENTIALS)
													error "Deployed VM's IP address is empty. Cannot copy build to test on the newly deployed VM. Exiting"											
												}
												for (String ipAddress : testVMIPList) 
												{											
													if(ipAddress && ipAddress.trim())
													{
														//Copy build to the deployed virtual machines for testing.
														echo '***************************************** COPYING BUILD TO THE DEPLOYED VM(s) FOR TESTING  *****************************************'
														echo "HCMX: Copying build to the virtual machine with IP address: $ipAddress"
														final String scpCMDOutput = sh(script: "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -rp ./build root@$ipAddress:/tmp/", returnStdout: true).trim()
													
													}
													else
													{
														echo "Deployed VM's IP address is empty. Cannot copy build to test VM"
														CancelSubscription(HCMX_SUB_CANCEL_DELAY_SECONDS, HCMX_SERVER_FQDN, HCMX_TENANT_ID, HCMX_PERSON_ID, subID, SMAX_AUTH_TOKEN, curlCMD, USE_PROXY, PROXY_REQUIRES_CREDENTIALS)
														error "Deployed VM's IP address is empty. Cannot copy build to test on the newly deployed VM. Exiting"
													}
												}
												
												for (String ipAddress : testVMIPList) 
												{											
																											
													// Test build on the deployed virtual machine.
													echo '***************************************** TESTING BUILD ON THE DEPLOYED/TEST VM(s) *****************************************'
													echo "HCMX: Testing build on the virtual machine with IP address: $ipAddress"
													final String remoteCMDOutput = sh(script: "ssh -o StrictHostKeyChecking=no root@$ipAddress /tmp/build/HelloWorld.sh", returnStdout: true).trim()
													
													// Validate test results from build execution results on remotely deployed virtual machine
													if(remoteCMDOutput && remoteCMDOutput.equals("Hello World"))
													{
														echo "Testing of new build was succesful.."															
													}
													else
													{
														echo "Testing of new build has failed... "
														CancelSubscription(HCMX_SUB_CANCEL_DELAY_SECONDS, HCMX_SERVER_FQDN, HCMX_TENANT_ID, HCMX_PERSON_ID, subID, SMAX_AUTH_TOKEN, curlCMD, USE_PROXY, PROXY_REQUIRES_CREDENTIALS)
														error "Testing of new build has failed..."
													}																										
												}
												echo "Deleting deployed VMs and then Proceeding to deploy stage."
												CancelSubscription(HCMX_SUB_CANCEL_DELAY_SECONDS, HCMX_SERVER_FQDN, HCMX_TENANT_ID, HCMX_PERSON_ID, subID, SMAX_AUTH_TOKEN, curlCMD, USE_PROXY, PROXY_REQUIRES_CREDENTIALS)
											}
											else
											{
												echo "Failed to get service instances from HCMX"
												CancelSubscription(HCMX_SUB_CANCEL_DELAY_SECONDS, HCMX_SERVER_FQDN, HCMX_TENANT_ID, HCMX_PERSON_ID, subID, SMAX_AUTH_TOKEN, curlCMD, USE_PROXY, PROXY_REQUIRES_CREDENTIALS)
												error "Failed to get service instances from HCMX"
											}
										} 
										else
										{
											echo "Failed to get subscription ID from HCMX"
											error "Failed to get subscription ID from HCMX"
										}
									}
									else
									{
										echo "Request to deploy new virtual machines has failed"
										error "Request to deploy new virtual machines has failed"
									}
								}
								else
								{
									echo "Unable to get user ID for the HCMX user to submit REST API calls to HCMX... "
									error "Unable to get user ID for the HCMX user to submit REST API calls to HCMX... "
								}
							}
							else
							{
								echo "Failed to get SMAX_AUTH_TOKEN"
								error "Failed to get SMAX_AUTH_TOKEN"
							}
						}
					}
                }
            }
        }
        stage('Deploy') 
		{
            steps 
			{
				/*  Deploy your new build to production environment.
					In this use case example, we do not deploy build to the production environment.
					This use case's main purpose is to demonstrate usage of HCMX to provision new test VMs, test new build on those test VMs and 
					delete those test VMs after testing is complete.
				*/
                
				echo '***************************************** DEPLOYING *****************************************'
            }
        }
    }
}

def CancelSubscription(int HCMX_SUB_CANCEL_DELAY_SECONDS, String HCMX_SERVER_FQDN, String HCMX_TENANT_ID, String HCMX_PERSON_ID, String subID, String SMAX_AUTH_TOKEN, String curlCMD, String USE_PROXY, String   PROXY_REQUIRES_CREDENTIALS)
{
	withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'ProxyCred', usernameVariable: 'PROXY_USER', passwordVariable: 'PROXY_USER_PSW']]) 
	{	
		// For demo and testing only. Comment out this line in production environment.
		echo "sleep for $HCMX_SUB_CANCEL_DELAY_SECONDS seconds before canceling subscription to delete deployed VM for testing."
		sleep(HCMX_SUB_CANCEL_DELAY_SECONDS)
		
		echo "HCMX: Canceling subscription"
		// Cancel subscription to delete deployed virtual machines. This frees up resources on cloud provider and reduces cloud spend.
		// Prepare HCMX cancel subscription URL using the subscription ID and the person ID that were obtained in earlier steps.
		final String HCMX_CANCEL_SUBSCRIPTION_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ess/subscription/cancelSubscription/" + HCMX_PERSON_ID + "/" + subID
		
		String subCancelResponse
		int subCancelResCode
		
		// Submit a REST API call to HCMX to cancel subscription, thereby delete deployed VM
		if (USE_PROXY.equalsIgnoreCase("NO") || ((USE_PROXY.equalsIgnoreCase("YES")) && (PROXY_REQUIRES_CREDENTIALS.equalsIgnoreCase("NO"))))
		{	
			(subCancelResponse, subCancelResCode)  = sh(script: '''set +x;''' + curlCMD + ''' -s -w '\\n%{response_code}' -X PUT "''' + HCMX_CANCEL_SUBSCRIPTION_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
		}
		else
		{
			(subCancelResponse, subCancelResCode)  = sh(script: '''set +x;''' + curlCMD + ''' --proxy-user $PROXY_USER:$PROXY_USER_PSW -s -w '\\n%{response_code}' -X PUT "''' + HCMX_CANCEL_SUBSCRIPTION_URL + '''" -k -H "Content-Type: application/json" -H "Accept: application/json" -H "Accept: text/plain" --cookie "TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN="''' + SMAX_AUTH_TOKEN + '''"" ''', returnStdout: true).trim().tokenize("\n")
		}
			
		if (subCancelResCode == 200) 
		{
			echo "HCMX Subscription canceled successfully"
		}
		else
		{
			echo "HCMX subscription cancellation failed. Removal of deployed VM has failed."
			echo "HCMX Subscription cancellation response: $subCancelResponse"
		}
	}
}
