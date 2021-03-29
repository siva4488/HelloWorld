pipeline 
{
    agent any
    environment 
	{
        // HCMX Server's fully qualified domain name
		HCMX_SERVER_FQDN = 'catvmlmpoc1.ftc.hpeswlab.net'
        
		// HCMX tenant's ID that has DND capability. DND capability is required to provision and manage VMs.
		// HCMX will be used to provision VMs on which testing of the new build will be performed.
		// After testing is complete, provisioned VMs are deleted so that expenses on public cloud is reduced and resource usage on private cloud is reduced.
		HCMX_TENANT_ID = '616409711'       
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
				
				echo 'Building...'
				sh 'mkdir build'
				sh 'cp HelloWorld.sh build/HelloWorld.sh'
				sh 'chmod 555 build/HelloWorld.sh'
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
				
				echo 'Testing..'
                script 
				{
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'HCMXUser', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) 
					{
                        final String HCMX_TENANT_ID = env.HCMX_TENANT_ID
                        final String HCMX_SERVER_FQDN = env.HCMX_SERVER_FQDN
						
						// HCMX REST APIs require SMAX AUTH TOKEN and TENANT ID to perform any POST, PUT and GET operations.
						// Build HCMX Authentication Token URL
                        final String HCMX_AUTH_URL = "https://" + HCMX_SERVER_FQDN + "/auth/authentication-endpoint/authenticate/token?TENANTID=" + HCMX_TENANT_ID
						
						// Submit a REST API call to HCMX to get SMAX_AUTH_TOKEN
                        final String SMAX_AUTH_TOKEN = sh(script: "curl -d '{\"login\":\"$USERNAME\",\"password\":\"$PASSWORD\"}' -X POST $HCMX_AUTH_URL -k --header \"Content-Type: application/json\"", returnStdout: true).trim()
						
						
						// Build HCMX Get Person ID URL
                        final String HCMX_GET_PERSON_ID_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ems/Person?filter=(Upn=%27" + USERNAME + "%27)&layout=Id"
						
						// Submit a REST API call to HCMX to get Person ID
                        final def (String personIDResponse, int personIDResCode)  = sh(script: "curl -s -w '\\n%{response_code}' \"$HCMX_GET_PERSON_ID_URL\" -k --header \"Content-Type: application/json\" -H \"Accept: application/json\" -H \"Accept: text/plain\" --cookie \"TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN=$SMAX_AUTH_TOKEN\"", returnStdout: true).trim().tokenize("\n")
						
						if (personIDResCode == 200) 
						{
							def personIDResponseJSON = new groovy.json.JsonSlurperClassic().parseText(personIDResponse)
							echo personIDResponse
							def HCMX_PERSON_ID = personIDResponseJSON.entities[0].properties.Id
							echo "HCMX Requested for person ID is $HCMX_PERSON_ID"
						
                        
							// Build HCMX create request URL
							final String HCMX_CREATE_REQUEST_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ess/request/createRequest"
							
							// Submit a REST API call to HCMX to deploy a new test server VM 
							final def (String depVMResponse, int depVMResponseCode) = sh(script: "curl -s -w '\\n%{response_code}' -X POST $HCMX_CREATE_REQUEST_URL -k --header \"Content-Type: application/json\" -H \"Accept: application/json\" -H \"Accept: text/plain\" --cookie \"TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN=$SMAX_AUTH_TOKEN\" -d '{\"entities\":[{\"entity_type\":\"Request\",\"properties\":{\"RequestedForPerson\":\"$HCMX_PERSON_ID\",\"StartDate\":1616452009778,\"RequestsOffering\":\"10096\",\"CreationSource\":\"CreationSourceEss\",\"RequestedByPerson\":\"$HCMX_PERSON_ID\",\"DataDomains\":[\"Public\"],\"UserOptions\":\"{\\\"complexTypeProperties\\\":[{\\\"properties\\\":{\\\"OptionSet0c6eb101a1a178c3c49c3badbc481f05_c\\\":{\\\"Option34c8d8d8403ac43361b8b8083004ef4a_c\\\":true},\\\"OptionSet2ee4a8f73fcd1606c1337172e8411e2a_c\\\":{\\\"Optionfda5ee32d7d24a63cb0035926c667e8b_c\\\":true},\\\"OptionSet473C6F2BE6F45DB8381664FC9097BE37_c\\\":{\\\"Option2E8493EA9AC2821929DA64FC90978A98_c\\\":true},\\\"changedUserOptionsForSimulation\\\":\\\"Optionad52a8efe1465faa8c389ae92bf90d0c_c&\\\",\\\"PropertyproviderId2E8493EA9AC2821929DA64FC90978A98_c\\\":\\\"2c908fac77eefca5017822299d726af6\\\",\\\"PropertydatacenterName2E8493EA9AC2821929DA64FC90978A98_c\\\":\\\"CAT\\\",\\\"PropertyvirtualMachine2E8493EA9AC2821929DA64FC90978A98_c\\\":\\\"catvmlmdep_t***CentOS 4/5 or later (64-bit)\\\",\\\"PropertycustomizationTemplateName2E8493EA9AC2821929DA64FC90978A98_c\\\":\\\"(Ts)catvmLinuxDHCP\\\",\\\"Optionfda5ee32d7d24a63cb0035926c667e8b_c\\\":true,\\\"Optionad52a8efe1465faa8c389ae92bf90d0c_c\\\":false}}]}\",\"Description\":\"<p>hello world test vm</p>\",\"RelatedSubscriptionName\":\"HelloWorldVM\",\"RelatedSubscriptionDescription\":\"<p>HelloWorldVM</p>\",\"RequestAttachments\":\"{\\\"complexTypeProperties\\\":[]}\",\"DisplayLabel\":\"Request: vCenter Compute - Deploy VM from Template\"}}],\"operation\":\"CREATE\"}'", returnStdout: true).trim().tokenize("\n")
							
							echo "Deploy a new test server VM request: HTTP response status code: $depVMResponseCode"
							
							if (depVMResponseCode == 200) 
							{
								def depVMResponseJSON = new groovy.json.JsonSlurperClassic().parseText(depVMResponse)
								echo depVMResponse
								def HCMX_REQUEST_ID = depVMResponseJSON.entity_result_list.entity[0].properties.Id
								echo "HCMX Request ID to deploy a new test server VM is $HCMX_REQUEST_ID"
								
								final String HCMX_GET_REQUEST_STATUS_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ems/Request?filter=Id=" + HCMX_REQUEST_ID + "\\&layout=PhaseId"
								println HCMX_GET_REQUEST_STATUS_URL
								String reqStatus = "Submitted"
								int reqCode = 0
								String reqResponse = "Nothing"
								while (reqStatus != 'Close')
								{
									println HCMX_GET_REQUEST_STATUS_URL
									(reqResponse, reqCode) = sh(script: "curl -s -w '\\n%{response_code}' $HCMX_GET_REQUEST_STATUS_URL -k --header \"Content-Type: application/json\" -H \"Accept: application/json\" -H \"Accept: text/plain\" --cookie \"TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN=$SMAX_AUTH_TOKEN\"", returnStdout: true).trim().tokenize("\n")
									echo "HTTP response status code: $reqCode"
									if (reqCode == 200) 
									{
										def reqResponseJSON = new groovy.json.JsonSlurperClassic().parseText(reqResponse)
										echo reqResponse
										reqStatus = reqResponseJSON.entities[0].properties.PhaseId
										echo "HCMX REQUEST status = $reqStatus"
										if (reqStatus.equalsIgnoreCase("Close"))
										{
											break;
										}
										else
										{
											echo "sleep for 30 seconds before checking status again"
											sleep(30)
										}                                        
									}
								}
								
								final String HCMX_GET_SUBSCRIPTION_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ems/Subscription?filter=(InitiatedByRequest=%27" + HCMX_REQUEST_ID + "%27%20and%20Status=%27Active%27)&layout=Id"
								println HCMX_GET_SUBSCRIPTION_URL
								final def (String subResponse, int subRescode)  = sh(script: "curl -s -w '\\n%{response_code}' \"$HCMX_GET_SUBSCRIPTION_URL\" -k --header \"Content-Type: application/json\" -H \"Accept: application/json\" -H \"Accept: text/plain\" --cookie \"TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN=$SMAX_AUTH_TOKEN\"", returnStdout: true).trim().tokenize("\n")
								if (subRescode == 200) 
								{
									def subResponseJSON = new groovy.json.JsonSlurperClassic().parseText(subResponse)
									echo subResponse
									subID = subResponseJSON.entities[0].properties.Id
									echo "HCMX Subscription ID = $subID" 
						
									final String HCMX_GET_SVCINSTANCE_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/cloud-service/getServiceInstance/" + subID
									println HCMX_GET_SVCINSTANCE_URL
									final def (String svcInstResponse, int svcInstRescode)  = sh(script: "curl -s -w '\\n%{response_code}' \"$HCMX_GET_SVCINSTANCE_URL\" -k --header \"Content-Type: application/json\" -H \"Accept: application/json\" -H \"Accept: text/plain\" --cookie \"TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN=$SMAX_AUTH_TOKEN\"", returnStdout: true).trim().tokenize("\n")
									if (svcInstRescode == 200) 
									{
										def svcInstResponseJSON = new groovy.json.JsonSlurperClassic().parseText(svcInstResponse)
										echo svcInstResponse
										def svcInstTopologyArray = svcInstResponseJSON.topology
										def ipAddress = ""

										for(def member : svcInstTopologyArray) 
										{
											if(member.type.name == 'CI_TYPE_SERVER') 
											{
												echo "this is server component"
												def svcInstPropertyArray = member.properties
												for(def propMember : svcInstPropertyArray) 
												{
													if(propMember.name == 'primary_ip_address') 
													{
														ipAddress = propMember.propertyValue
														echo "IP address is $ipAddress"													 
														break
													}
												}
												break
											}
										}
										
										echo "IP address is $ipAddress"	
										final String scpCMDOutput = sh(script: "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -rp ./build root@$ipAddress:/tmp/", returnStdout: true).trim()
										println scpCMDOutput
										final String remoteCMDOutput = sh(script: "ssh -o StrictHostKeyChecking=no root@$ipAddress /tmp/build/HelloWorld.sh", returnStdout: true).trim()
										
										sleep(60)
										
										
										final String HCMX_CANCEL_SUBSCRIPTION_URL = "https://" + HCMX_SERVER_FQDN + "/rest/" + HCMX_TENANT_ID + "/ess/subscription/cancelSubscription/" + HCMX_PERSON_ID + "/" + subID
										println HCMX_CANCEL_SUBSCRIPTION_URL
										final def (String subCancelResponse, int subCancelRescode)  = sh(script: "curl -s -w '\\n%{response_code}' -X PUT \"$HCMX_CANCEL_SUBSCRIPTION_URL\" -k --header \"Content-Type: application/json\" -H \"Accept: application/json\" -H \"Accept: text/plain\" --cookie \"TENANTID=$HCMX_TENANT_ID;SMAX_AUTH_TOKEN=$SMAX_AUTH_TOKEN\"", returnStdout: true).trim().tokenize("\n")
										if (subCancelRescode == 200) 
										{
											def subCancelResponseJSON = new groovy.json.JsonSlurperClassic().parseText(subCancelResponse)
											echo subCancelResponse					                                                                         
										}
										echo subCancelResponse
																									
										if(remoteCMDOutput == "Hello World")
										{
											echo "Testing of new build was succesful.. Proceeding to deploy stage."
										}
										else
										{
											echo "Testing of new build has failed... "
											error 'Testing of new build has failed...'
										}								
									}
								}              
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
                
				echo 'Deploying...'
            }
        }
    }
}