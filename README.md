# Initial page

## Send Email:

System Configuration:

> app/code/VENDOR/MODULE/etc/adminhtml/system.xml

```markup
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="ean" translate="label" type="text" sortOrder="1" showInDefault="1" showInWebsite="1" showInStore="1">
            <class>separator-top</class>
            <label>NAME</label>
            <tab>ID</tab>
            <resource>VENDOR_MODULE::CONFIG_NAME</resource>
            <group id="emails" translate="label" type="text" sortOrder="1" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>Config Send Email</label>
                <field id="enabled" translate="label" type="select" sortOrder="0" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Enabled</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="sender_name" translate="label" type="text" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Sender Name</label>
                    <depends>
                        <field id="enabled">1</field>
                    </depends>
                </field>
                <field id="sender_email" translate="label" type="text" sortOrder="20" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Sender Email</label>
                    <depends>
                        <field id="enabled">1</field>
                    </depends>
                </field>
                <field id="template" translate="label" type="select" sortOrder="30" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Email Template:</label>
                    <comment>Email template chosen based on theme fallback when "Default" option is selected.</comment>
                    <source_model>Magento\Config\Model\Config\Source\Email\Template</source_model>
                    <depends>
                        <field id="enabled">1</field>
                    </depends>
                </field>
                <field id="copy_to" translate="label comment" type="text" sortOrder="40" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Email Copy To</label>
                    <depends>
                        <field id="enabled">1</field>
                    </depends>
                </field>
            </group>
        </section>
    </system>
</config>
```

Default email template:

> app/code/VENDOR/MODULE/view/frontend/email/email\_template.html

```markup
{{template config_path="design/email/header_template"}}

//Detail here

{{template config_path="design/email/footer_template"}}
```

Set default config for template email:

> app/code/VENDOR/MODULE/etc/email\_templates.xml

```markup
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Email:etc/email_templates.xsd">
    <template id="ean_emails_template" label="Default Email" file="email_template.html" type="html" module="VENDOR_MODULE" area="frontend"/>
</config>
```

{% hint style="info" %}
template id="ean\_emails\_template" =&gt; get id from config: **"sectionid"**_**\_**_**"group\_id"\_"field\_id"**
{% endhint %}





