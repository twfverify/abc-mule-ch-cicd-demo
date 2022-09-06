# abc-mule-ch-cicd-demo

<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:vm="http://www.mulesoft.org/schema/mule/vm"
	xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core" xmlns:kafka="http://www.mulesoft.org/schema/mule/kafka"
	xmlns:http="http://www.mulesoft.org/schema/mule/http"
	xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="
http://www.mulesoft.org/schema/mule/vm http://www.mulesoft.org/schema/mule/vm/current/mule-vm.xsd http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd
http://www.mulesoft.org/schema/mule/kafka http://www.mulesoft.org/schema/mule/kafka/current/mule-kafka.xsd
http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd">
	<http:listener-config name="HTTP_Listener_config" doc:name="HTTP Listener config" doc:id="f1e9f03c-0606-410a-a56b-45f426e7373d" >
		<http:listener-connection host="0.0.0.0" port="8081" />
	</http:listener-config>
	<vm:config name="VM_Config" doc:name="VM Config" doc:id="f4c30da4-432b-493f-9614-906826c053a6" >
		<vm:connection />
		<vm:queues >
			<vm:queue queueName="party-insert-Q" queueType="PERSISTENT" />
			<vm:queue queueName="party-update-Q" queueType="PERSISTENT" />
			<vm:queue queueName="return-Q" queueType="PERSISTENT" />
		</vm:queues>
	</vm:config>
	<kafka:producer-config name="Apache_Kafka_Producer_configuration" doc:name="Apache Kafka Producer configuration" doc:id="548296c5-6f13-486c-a2cd-0a2443866e80" >
		<kafka:producer-plaintext-connection >
			<kafka:bootstrap-servers >
				<kafka:bootstrap-server value="localhost:9092" />
			</kafka:bootstrap-servers>
		</kafka:producer-plaintext-connection>
	</kafka:producer-config>
	<kafka:consumer-config name="Apache_Kafka_Consumer_configuration" doc:name="Apache Kafka Consumer configuration" doc:id="e96b8579-ce09-4045-b405-3944f0155725" >
		<kafka:consumer-plaintext-connection autoOffsetReset="EARLIEST" groupId="test-grp">
			<kafka:bootstrap-servers >
				<kafka:bootstrap-server value="localhost:9092" />
			</kafka:bootstrap-servers>
			<kafka:topic-patterns >
				<kafka:topic-pattern value="demo1" />
			</kafka:topic-patterns>
		</kafka:consumer-plaintext-connection>
	</kafka:consumer-config>
	<flow name="kafka-testFlow" doc:id="2db2b66c-65e7-42fd-8c3d-b5b0470df5d9" >
		<http:listener doc:name="Listener" doc:id="d06287a5-7123-4cf1-bf3d-e8cd418df706" config-ref="HTTP_Listener_config" path="/multi-publish"/>
		<foreach doc:name="For Each" doc:id="9bea3a36-c821-4fd1-90e1-525365578fe1" collection="#[payload]">
			<kafka:publish doc:name="Publish" doc:id="5cdaf002-116a-45db-9f79-e481c0a8fdde" key="#[now()]" topic="demo1" config-ref="Apache_Kafka_Producer_configuration" />
		</foreach>
		<ee:transform doc:name="Transform Message" doc:id="2aed259c-4692-4169-ac4d-6f90c1b1ab7b" >
			<ee:message >
				<ee:set-payload ><![CDATA[%dw 2.0
output application/json
---
{
	"message":"message published"
}]]></ee:set-payload>
			</ee:message>
		</ee:transform>
	</flow>
	<flow name="kafka-testFlow5" doc:id="100d747f-0b28-4307-9227-525ac235964d" >
		<http:listener doc:name="Listener" doc:id="73eb3b97-268c-4c75-8e03-127bd8c9c0ab" config-ref="HTTP_Listener_config" path="/single-publish"/>
		<kafka:publish doc:name="Publish" doc:id="bfb66fc5-bde7-4db7-9500-bd49e1598707" config-ref="Apache_Kafka_Producer_configuration" topic="demo1" key="#[now()]"/>
		<ee:transform doc:name="Transform Message" doc:id="1d77fc30-6755-4737-ba13-108e8f701081" >
			<ee:message >
				<ee:set-payload ><![CDATA[%dw 2.0
output application/json
---
{
	"message":"message published"
}]]></ee:set-payload>
			</ee:message>
		</ee:transform>
	</flow>
	<flow name="kafka-testFlow2" doc:id="8dd63652-03e8-4785-9fcd-79407e0a8873" >
		<scheduler doc:name="Scheduler" doc:id="867cd558-fd53-41ee-b160-40bec06b8e32" >
			<scheduling-strategy >
				<fixed-frequency frequency="60" timeUnit="SECONDS"/>
			</scheduling-strategy>
		</scheduler>
		<set-variable value="#[[]]" variableName="result" />
		<try>
			<foreach collection="#[1 to 10]">
				<kafka:consume doc:name="Consume" doc:id="9aa28986-6912-426b-ada7-33495d825982" config-ref="Apache_Kafka_Consumer_configuration"/>
				<ee:transform doc:name="Transform Message" doc:id="874da541-7b77-4487-90ad-5b516fe30c90">
			<ee:message>
				<ee:set-payload><![CDATA[%dw 2.0
output application/json
---
read(payload,"application/json")]]></ee:set-payload>
			</ee:message>
		</ee:transform>
				<ee:transform doc:name="Transform Message" doc:id="7e60d537-9b5e-470a-8e27-23193f2ccd72" >
					<ee:message >
						<ee:set-payload ><![CDATA[%dw 2.0
output application/json
---
{
	"level" : if(payload.payload.mids != null) "OUTLET" else
	          if(payload.payload.ultimateParentPartyId == payload.payload.partyId) "GROUP" else
	          "COMPANY",	          
	"meta" : payload.meta,
	"payload" : payload.payload
}]]></ee:set-payload>
					</ee:message>
				</ee:transform>
				<ee:transform doc:id="3ee48104-0c60-4bda-b5f3-ce582948982c">
					<ee:message>
					</ee:message>
					<ee:variables>
						<ee:set-variable variableName="result"><![CDATA[%dw 2.0
output application/java
---
vars.result << payload]]></ee:set-variable>
					</ee:variables>
				</ee:transform>
			</foreach>
			<error-handler>
				<on-error-continue enableNotifications="true" logException="false" type="ANY" >
				</on-error-continue>
			</error-handler>
		</try>
		<set-payload value="#[vars.result]" doc:name="Set Payload" doc:id="ba70f273-85d2-4d91-987e-d3791ea18c94" />
		<set-variable value="#[sizeOf(payload)]" doc:name="Set Variable" doc:id="d9e56511-e9fe-4824-815f-d76fa1f9f2fa" variableName="sizeOfPayload" />
		<choice doc:name="Choice" doc:id="cae2a756-6e15-4352-a15d-f38465d66ea1" >
			<when expression="#[vars.sizeOfPayload &gt; 0]">
				<flow-ref doc:name="Flow Reference" doc:id="6bf6beeb-fdd6-4c32-8c9f-4c55694c4ea9" name="kafka-testSub_Flow"/>
			</when>
			<otherwise >
				<logger level="INFO" doc:name="Logger" doc:id="baf30a16-adc6-4efa-9851-8024c5cc52db" message="no message found" />
			</otherwise>
		</choice>
	</flow>
	<sub-flow name="kafka-testSub_Flow" doc:id="bc311d61-d57d-4df9-8546-54346511314b" >
		<ee:transform doc:name="Transform Message" doc:id="54737108-b09d-4b5f-945d-8b39dc46e5f8">
			<ee:message>
				<ee:set-payload><![CDATA[%dw 2.0
output application/json
---
payload groupBy ((item, index) -> item.payload.ultimateParentPartyId) 
mapObject ((value, key, index) -> {
    (key) : (value orderBy ((item, index) -> item.meta.occurred_at))
})
pluck ((value, key, index) -> value)]]></ee:set-payload>
			</ee:message>
		</ee:transform>
		<foreach doc:name="For Each" doc:id="218c4c29-976a-421b-b5ad-901d03734036" collection="#[payload]">
			<ee:transform doc:name="Transform Message" doc:id="9b5771f3-b2a8-4eea-b791-19e8e6c1fd04" >
				<ee:message />
				<ee:variables >
					<ee:set-variable variableName="groupPayload" ><![CDATA[%dw 2.0
output application/json
---
payload filter ((item, index) -> item.level == "GROUP" )]]></ee:set-variable>
					<ee:set-variable variableName="companyPayload" ><![CDATA[%dw 2.0
output application/json
---
payload filter ((item, index) -> item.level == "COMPANY" )]]></ee:set-variable>
					<ee:set-variable variableName="outletPayload" ><![CDATA[%dw 2.0
output application/json
---
payload filter ((item, index) -> item.level == "OUTLET" )]]></ee:set-variable>
				</ee:variables>
			</ee:transform>
			<choice doc:name="Choice" doc:id="50d7d10f-5d69-46ba-9497-264b101181ee" >
				<when expression='#[isEmpty(vars.outletPayload)]'>
					<set-payload value="#[vars.groupPayload ++ vars.companyPayload]" doc:name="Set Payload" doc:id="4e7ccbca-5013-4241-8b13-6d4996d057e5" />
					<foreach doc:name="For Each" doc:id="6cd5596b-2cfd-4d71-8c35-4a5d4d2a9800" collection="#[payload]" >
						<ee:transform doc:name="Transform Message" doc:id="b380c95d-3a4b-4cd6-97b4-ae7d7b31fa9a" >
							<ee:message >
								<ee:set-payload ><![CDATA[%dw 2.0
output application/json
---
{	   
	"meta" : payload.meta,
	"payload" : payload.payload
}]]></ee:set-payload>
							</ee:message>
						</ee:transform>
						<kafka:publish doc:name="Publish" doc:id="731050d7-23f4-401e-af59-b117a8961249" config-ref="Apache_Kafka_Producer_configuration" topic="demo1" key="#[now()]" />
					</foreach>
					<logger level="INFO" doc:name="Logger" doc:id="59b07493-81a4-457a-b510-4000b10460d1" message="re publish done" />
				</when>
				<when expression="#[!isEmpty(vars.outletPayload) and isEmpty(vars.groupPayload)]">
					<set-payload value="#[vars.outletPayload]" doc:name="Set Payload" doc:id="83d46ab4-30f3-4b35-a8be-969297db935b" />
					<vm:publish doc:name="Publish" doc:id="be8e91bc-f4bf-421b-8476-1f7f10d10370" config-ref="VM_Config" sendCorrelationId="AUTO" queueName="party-update-Q"/>
				</when>
				<otherwise >
				    <!--    
				    <set-payload value="#[vars.groupPayload ++ vars.companyPayload ++ vars.outletPayload]" doc:name="Set Payload" doc:id="00a4b555-1b9c-42f9-a9ce-365bc8298444" />
					<vm:publish queueName="party-insert-Q" doc:name="Publish" doc:id="42eac529-eb4e-4053-bdf5-562ad3ec0364" config-ref="VM_Config" sendCorrelationId="AUTO" />
					
					-->
					   
					<foreach doc:name="For Each" doc:id="554ec77d-bd51-4b72-8cb0-c4bafec72321" collection="#[vars.companyPayload]">
						<set-variable value="#[payload.payload.partyId]" doc:name="Set Variable" doc:id="845454ad-1af6-4a47-bd3a-c054ec5b1a4b" variableName="companyId"/>
						<ee:transform doc:name="Transform Message" doc:id="879d7841-3876-4b4d-9b6e-d84c7f6e454c">
							<ee:message>
							</ee:message>
							<ee:variables >
								<ee:set-variable variableName="group_rec" ><![CDATA[%dw 2.0
output application/json
---
(vars.groupPayload)[0]]]></ee:set-variable>
								<ee:set-variable variableName="company_rec" ><![CDATA[%dw 2.0
output application/json
---
payload]]></ee:set-variable>
							</ee:variables>
						</ee:transform>
						<set-variable value="#[[]]" doc:name="Set Variable" doc:id="751b79b5-e2c6-4f08-a8d1-5d4b03252618" variableName="outletList"/>
						<foreach doc:name="For Each" doc:id="1d99af0b-b748-4df2-b120-3e845ca051e5" collection="#[vars.outletPayload]">
							<choice doc:name="Choice" doc:id="b3500efa-b6eb-49d7-aed2-6892361c5acc" >
								<when expression="#[payload.payload.parentPartyId == vars.companyId]">
									<ee:transform doc:name="Transform Message" doc:id="aafaa91a-e5bb-4ad4-b0be-cd008d23de7a" >
										<ee:message >
										</ee:message>
										<ee:variables >
											<ee:set-variable variableName="outletList" ><![CDATA[%dw 2.0
output application/json
---
vars.outletList << payload]]></ee:set-variable>
										</ee:variables>
									</ee:transform>
								</when>
							</choice>
						</foreach>
						<set-payload value="#[vars.outletList]" doc:name="Set Payload" doc:id="295d65fd-be9e-4fef-b79e-953941f0af6f" />
						<logger level="INFO" doc:name="Logger" doc:id="8687ab05-1c2c-4c8d-975f-f1ad52ac9499" message="#[payload]" />
						<ee:transform doc:name="Transform Message" doc:id="290fd0c3-29ea-4e96-941d-d53fda9982c5" >
							<ee:message >
								<ee:set-payload ><![CDATA[%dw 2.0
output application/json
---
vars.group_rec ++ vars.company_rec ++ payload]]></ee:set-payload>
							</ee:message>
						</ee:transform>
						<logger level="INFO" doc:name="Logger" doc:id="24881ce0-e7c8-4286-8b9d-6d0400c45855" message="#[payload]"/>
						<vm:publish queueName="party-insert-Q" doc:name="Publish" doc:id="e48d3011-cdbb-4ef0-89b7-79c808a446f4" config-ref="VM_Config" sendCorrelationId="AUTO" />
						<remove-variable doc:name="Remove Variable" doc:id="6f875a95-f136-4a3c-b9d9-139b5e36d7ba" variableName="#[vars.publishPayload]"/>
				
					</foreach>
					
				</otherwise>						
		</choice>
		</foreach>
		
	</sub-flow>
	<flow name="kafka-testFlow1" doc:id="c37abaf7-bf92-4a2a-bd66-9132e6faae24" >
		<vm:listener doc:name="Listener" doc:id="c08bbe95-c9bb-4cb1-91b3-19401133c3f5" config-ref="VM_Config" queueName="party-insert-Q"/>
		<logger level="INFO" doc:name="Logger" doc:id="b69dd3ca-5f3f-48a2-8902-4559d6ba3df7" message="log message insert "/>
	</flow>
	<flow name="kafka-testFlow3" doc:id="ed43306a-869b-48ca-ad8e-89197d5bf343" >
		<vm:listener queueName="party-update-Q" doc:name="Listener" doc:id="a4b72d9c-c820-4658-9c02-6bede00a8ec5" config-ref="VM_Config"/>
		<logger level="INFO" doc:name="Logger" doc:id="1c31ab61-3b3b-4bcd-ab6d-08634a9adaee" message="log messgae update"/>
	</flow>
</mule>



{
	"meta": {
		"category" : "event",
		"version" : "1.0",
		"occurred_at": "2022-08-24T15:51:02"	
	},
	"payload" : {
		"partyId":"PO4000000008",
		"ultimateParentPartyId":"PO4000000008",
		"parentPartyId": "PO4000000008",
		"legalName":"Group",
		"tradingName" : "Group Example",
		"boardingStatus" : "trading-live",
		"mids" : null
	}
},
{
	"meta": {
		"category" : "event",
		"version" : "1.0",
		"occurred_at": "2022-08-24T15:51:05"	
	},
	"payload" : {
		"partyId":"PO4000000009",
		"ultimateParentPartyId":"PO4000000008",
		"parentPartyId": "PO4000000008",
		"legalName":"Company",
		"tradingName" : "Company Example",
		"boardingStatus" : "trading-live",
		"mids" : null
	}
},
{
	"meta": {
		"category" : "event",
		"version" : "1.0",
		"occurred_at": "2022-08-24T15:51:05"	
	},
	"payload" : {
		"partyId":"PO4000000010",
		"ultimateParentPartyId":"PO400000008",
		"parentPartyId": "PO4000000008",
		"legalName":"Company",
		"tradingName" : "Company Example",
		"boardingStatus" : "trading-live",
		"mids" : null
	}
},
{
	"meta": {
		"category" : "event",
		"version" : "1.0",
		"occurred_at": "2022-08-24T15:51:09"	
	},
	"payload" : {
		"partyId":"PO4000000011",
		"ultimateParentPartyId":"PO4000000008",
		"parentPartyId": "PO4000000009",
		"legalName":"Outlet 1",
		"tradingName" : "Outlet 1 Example",
		"boardingStatus" : "trading-live",
		"mids" : ["65349103"]
	}
},
{
	"meta": {
		"category" : "event",
		"version" : "1.0",
		"occurred_at": "2022-08-24T15:51:09"	
	},
	"payload" : {
		"partyId":"PO4000000012",
		"ultimateParentPartyId":"PO4000000008",
		"parentPartyId": "PO4000000010",
		"legalName":"Outlet 1",
		"tradingName" : "Outlet 1 Example",
		"boardingStatus" : "trading-live",
		"mids" : ["65349103"]
	}
}
]
