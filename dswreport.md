# DSW Reports � ��������� ������� DeepSeeWeb

�������� [InterSystems Data Platform](https://www.intersystems.com/resources/detail/intersystems-iris-data-platform/) �������� � ���� ������������� ������ [DeepSee](https://www.intersystems.com/products/intersystems-iris/analytics/), ��������������� ����������� ��������� ������������� ��������. �������� ����� ���������� �������������� ��� ������ ���������� [DeepSeeWeb](https://github.com/intersystems-ru/DeepSeeWeb) � ������������ ������ �������������, �������� [���������](https://analytics.community.intersystems.com/) [InterSystems Developer Community](https://community.intersystems.com/). �� ������ ��� ������� ������ ����� ����������� ������������ � �������������� ������������ ������� �� ������������ ��������, ������������� � ��������� ���������. ����� �������� ��� ������ � PDF � ���������� �� ����������� �����, ���������� �� ����������. ��� � �������� ������ DSW Reports, ������� ��������� ���������� DeepSeeWeb, � ������ ������ ����������� ����. � ���� ������ ����� ������� ��� ������������ DSW Reports.
<cut />
## ��� ����� DSW Reports?
[DSW Reports](https://github.com/intersystems-community/dsw-reports) - ��� ���������� DSW, ���������� �� AngularJS, ������� ��������� ������� ���������� ��� �������������� ��������� �������. DSW Reports ������������ [DeepSeeWeb](https://github.com/intersystems-ru/DeepSeeWeb) ��� ��������� �������� � [MDX2JSON](https://github.com/intersystems-ru/Cache-MDX2JSON) ��� ��������� MDX ��������.

### �����������:
- ��������� ��������� �������� � �������������� ���������.
- ����� ����������� ���������� ������������ MDX ��������.
- �������������� �������� PDF ������� � �������� �� �� �����
- ������������ �������� ���� ������ ��� ������ CSS ������

![HTML report](https://raw.githubusercontent.com/MakarovS96/images/master/Report.png)

## �������� ������
��� ������������ ������ � DSW Reports ���������� ������� ��� ������� 2 �����: 

- **index.html** - ������ � ������� �������� ������, ������ �� ����������.
- **config.js** - ������������  ������, �������� ��� ������ �������, �������� �� ���������� ������.

� ����� ������������ ����� ������ ����������� ����������� ������� **getConfiguration**.
```javascript
// ����� ��������� ������
function getConfiguration(params){...}
```
������� **getConfiguration** ��������� ������ *params*, ������� �������� � ���� ��������� �� ������ URL � �������������� �������� "***server***", ���������� ������� �������. �������� "***server***" ����� ���: `protocol://host:port`.

��������� ������� *params* ����� ���������� � ����� ����� ������ ����� URL ������. ��������, ���� ��������� �������� ������� �������� �� �������, ����� ������� � URL �������� "***filter***" � �� ����� �������� ����� ������ *params*.
```javascript
//<protocol://host:port>/dsw/reports/report_dir/index.html?filter=NOW
function getConfiguration(params){
    var filter = params["filter"]; // filter = "NOW"
}
```

������� **getConfiguration** ���������� ������, ���������� 3 ��������:

- *REPORT_NAME* - �������� ������
- *BLOCKS* - ������ ������ ������
- *NAMESPACE* - ������� � ������� ��� ������

���������� ��������� ������ ������ *BLOCKS*. ���� - ������ � ����������� �������, ����������� ����������� ����� � �.�.

### ��� �����:
```javascript
{
    "title": String,            //��������� �����
    "note": String,             //��������� ��� ������. ����� ��������� HTML ���
    "widget": {                 //��������� iframe �������:
        "url": String,          //URL ��������� ��� iframe
        "height": Number,       //������ iframe
        "width": Number         //������ iframe
    },
    "totals":[{                 //��������� �������� ����������� � ������� MDX
        "mdx": String           //MDX ������
        "strings": [{           //������ �������� �� �������
            "title": String,    //��������� ������. ����� ������������ HTML.
            "value": String,    //�������� ������ �� ���������
            "value_append": String, //������� ��� ��������. 
                                //����� �������������� ��� ������ %, $ � �.�. 
                                //% ����������� �������� � ���������� (x * 100).
                                //����� ������������ HTML.
            "row": Number       //����� ������ MDX �������, 
                                //�� ������� ������ ��������. 
                                //�� ��������� 0.
        },{...}]
    },{...}]}
```
��� ���� �����������, ���� ���� �� ����� ����� ��������� ��� ������ �������.

<details>
<summary>������ �����</summary>
```javascript
{
     title: "Persons",
     note: "",
     widget: {
        url: server + "/dsw/index.html#!/d/KHAB/Khabarovsk%20Map.dashboard" + 
        "?widget=1&height=420&ns=" + namespace,
        width: 700,
        height: 420
     }
}
```
</details>

<details>
<summary>��� ������</summary>
```javascript
{
    title: "Khabarovsky krai",
    note: "Something note (only static)",
    widget: {
        url: server + "/dsw/index.html#!/d/KHAB/Khabarovsk%20Map.dashboard" + 
        "?widget=0&height=420&isLegend=true&ns=" + namespace,
        width: 495,
        height: 420
    },
    totals: [{
       mdx: "SELECT NON EMPTY " + 
      "[Region].[H1].[Region].CurrentMember.Properties(\"Population\") ON 0,"+
      "NON EMPTY {[Region].[H1].[Region].&[���������]," + 
      "[Region].[H1].[Region].&[�����������-��-�����],"+
      "[Region].[H1].[Region].&[������������� �����]} ON 1 FROM [KHABCUBE]",
       strings: [{
            title: "Khabarovsk: ",
            value: "None",
            value_append: " ���."
        }, {
            title: "Komsomolsk-on-Amur: <br />",
            value: "None",
            value_append: " ���.",
            row: 1
        }, {
            title: "Komsomolsky district: <br />",
            value: "None",
            value_append: " ���.",
            row: 2
        }]
    }]
}
```
</details>

### ��� ��������� ����?
�������� ���� ��� ���������� � ����� - ��� **url** ��� �������� ������� � **mdx** ��� �������� ����������� ��������.   
- **MDX** ����� ��������� � �������, �� ������������� ������ ��� ��� ������ ����������� ����������� [Analyzer](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=D2ANLY_ch_intro), ����������� � DeepSee.
![Analyzer](https://raw.githubusercontent.com/MakarovS96/images/master/Analyzer.png)
- **URL** ����� �������� ��� ������ DeepSeeWeb. ������� ���������� � ����� ��� �������� *iframe*, ����������� ������� �������� ������� DeepSeeWeb. ��� ����, ����� �������� ������ �� �������� ���� ������� ����� *"Share"* � ����������� ���� �������.
![Share](https://raw.githubusercontent.com/MakarovS96/images/master/Share.png)

### ������������ �������� ���� ������. 
������ � ������������ ������ ������������ ���� **style.css** ����������� ������������� ������� ��� ������. � ��� ���������� ���������� ����� �������� ����������� ����� ���������� ������. ����� ����� ��������� ���� ������ ������ � ������������ �� � ����� **index.html**.

## �������� �� E-mail

�������� ����� ��� ����� � �������� � ����� ������� � DeepSeeWeb. �.�. ������������� HTML ����� ������ �������� �� ������. ��� ����� ������� �����, �������������� ��� � PDF � ��������� �� �����? ��� ������������� ������� [pthantomjs](http://phantomjs.org/) � ���������� SMTP ������. ��� ���������� � ��������� phantomjs ����� ���������� ����� ([windows](https://youtu.be/L8Lw53MjDdY), [ubuntu](https://www.vultr.com/docs/how-to-install-phantomjs-on-ubuntu-16-04)). �� ��� ����� ���� ��������� SMTP ������ � ������� ������� � [��������� �����](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=GSA_manage_taskmgr). 

### ��������� SMTP
��� ��������� ������������ � ���������.
1. ������� ���� ��������� ����� ��� ��������
```
// ������� ��� ��������� SMTP
do ##class(DSW.Report.EmailSender).setConfig(server, port, username, 
                                                 password, sender, SSLConfig)
```
- **server** - ����� SMTP �������.  
- **port** - ���� ��� ��������� ��������.  
- **username** � **password** - ������������������ ������.  
- **sender** - E-mail ����� ��������.  
- **SSLConfig** - *�����������*. ��� [SSL ������������](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=GCAS_ssltls).   
2. ����� ������� ��������� ������ ������������� ��� ��������
```
// ������� ��� ���������� ������������
do ##class(DSW.Report.EmailSender).addRecipient(email)
// ������� ��� �������� ������������
do ##class(DSW.Report.EmailSender).deleteRecipient(email)
```
3. ����� ���������� ����� ����� ��������� ��������
```
// ������� ����������� ��������
do ##class(DSW.Report.Task).Run(url, reportname)
```
- **url** - ������ �� �����.  
- **reportname** - �������� ������. ������������ ��� ��������� PDF.

### �������������� ������ ��������
��� ������������� �������� ������������� [���������� �����](https://docs.intersystems.com/latest/csp/docbook/DocBook.UI.Page.cls?KEY=GSA_manage_taskmgr). �������� ����� ������ �� ���������� �����������:
1. �� ������ �������� ������������� ������� ������� � ����������� ���� ������� ��� ������� ��������.
![Task1](https://raw.githubusercontent.com/MakarovS96/images/master/Task1.png)
2. �� ������ �������� ������������� ����� � ������������� ������� ������.
![Task2](https://raw.githubusercontent.com/MakarovS96/images/master/Task2.png)
3. ��������� ����� ���� ������ *"���������"*.

��, ����� ���� ���� ����������� � ��� ��������� ���������������� �����, ��������� �� �������� DeepSeeWeb, ������� � �������� ����� ����������� �� ����� � ���� PDF.

������ �������� ������ ����� ���������� [�����](https://bit.ly/2MhMLfh).
���� ������������ ����� ������ ���������� [�����](https://github.com/intersystems-community/dc-analytics/blob/master/src/reports/week/config.js).
� [�����](https://community.intersystems.com/post/analysing-developer-community-activity-using-intersystems-analytics-technology-deepsee) ����� ����������� �� ������������ �������� �������.
[������](https://github.com/intersystems-community/dsw-reports) �� �����������.