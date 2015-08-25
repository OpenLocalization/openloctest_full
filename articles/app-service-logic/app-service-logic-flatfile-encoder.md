<properties 
   pageTitle="BizTalk Flat File Encoder" 
   description="BizTalk Flat File Encoder" 
   services="app-service\logic" 
   documentationCenter=".net,nodejs,java" 
   authors="rajram" 
   manager="dwrede" 
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration" 
   ms.date="07/01/2015"
   ms.author="rajram"/>

# BizTalk Flat File Encoder

BizTalk Flat File Encode Decode connector helps your app interop between flat file data (example excel, csv) and XML data. It can convert a given flat file instance to XML and vice versa.

##Using the BizTalk Flat File Encoder
1. To use the BizTalk Flat File Encoder, you need to first create an instance of the BizTalk Flat File Encoder API app. This can be done either inline while creating a logic app or by selecting the BizTalk Flat File Encoder API app from the Azure Marketplace.

###Configure BizTalk Flat File Encoder
BizTalk Flat File Encoder takes schemas as part of its configuration. Users can launch the API App configuration blade by either launching the API App directly from the Azure portal, or by double-clicking the API App on the designer surface.

![BizTalk Flat File Encoder Configuration][1]

In the API App blade, users can configure schemas by clicking on *Schemas* part.

![BizTalk Flat File Encoder Schemas Part][2]

Users can upload schemas from disk or generate one from a flat file instance or a JSON instance.

![BizTalk Flat File Encoder Schemas Part][3]


###Using the BizTalk Flat File Encoder in design surface
Once configured, users can click on *->* and choose an action from a list of actions.

![BizTalk Flat File Encoder List of actions][4]

####Flat File to Xml

![BizTalk Flat File Encoder List of actions][5]

<table>
	<tr>
		<th>Parameter</th>
		<th>Type</th>
		<th>Description of the parameter</th>
	</tr>
	<tr>
		<td>Flat File</td>
		<td>string</td>
		<td>Content of the input flat file</td>
	</tr>
	<tr>
		<td>Schema Name</td>
		<td>string</td>
		<td>Name of the schema which represents the input flat file</td>
	</tr>
	<tr>
		<td>Root Name</td>
		<td>string</td>
		<td>Root node name of the flat file schema</td>
	</tr>
</table>


The action returns the output as a string - Output Xml. Output Xml contains the xml representation of the input flat file content.

####Xml to Flat File

![BizTalk Flat File Encoder List of actions][6]

<table>
	<tr>
		<th>Parameter</th>
		<th>Type</th>
		<th>Description of the parameter</th>
	</tr>
	<tr>
		<td>Input Xml</td>
		<td>string</td>
		<td>Input Xml content</td>
	</tr>
</table>

The action returns the output as a string - Flat File. Output contains the flat file representation of the input xml content.

<!-- References -->
[1]: ./media/app-service-logic-flatfile-encoder/FlatFileEncoder.ClickToConfigure.PNG
[2]: ./media/app-service-logic-flatfile-encoder/FlatFileEncoder.SchemasPart.PNG
[3]: ./media/app-service-logic-flatfile-encoder/FlatFileEncoder.SchemaUpload.PNG
[4]: ./media/app-service-logic-flatfile-encoder/FlatFileEncoder.ListOfActions.PNG
[5]: ./media/app-service-logic-flatfile-encoder/FlatFileEncoder.FlatFileToXml.PNG
[6]: ./media/app-service-logic-flatfile-encoder/FlatFileEncoder.XmlToFlatFile.PNG
 
