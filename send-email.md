# Send Email

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
<!--@subject {{trans "Email Subject"}}  @-->
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
template id="ean\_emails\_template" =&gt; get id from config: "sectionid"\_"group\_id"\_"field\_id"
{% endhint %}

Code send email:

```php
<?php

namespace SkyPremium\Ean\Helper;

/**
 * Class SendEmail
 * @package SkyPremium\Ean\Helper
 */
class SendEmail extends \Magento\Framework\App\Helper\AbstractHelper
{
    /**
     *
     */
    const XML_CONFIG_PATH = 'ean/emails/';

    /**
     * @var \Magento\Store\Model\StoreManagerInterface
     */
    protected $_storeManager;

    /**
     * @var \Magento\Framework\Translate\Inline\StateInterface
     */
    protected $inlineTranslation;

    /**
     * @var \Magento\Framework\Mail\Template\TransportBuilder
     */
    protected $_transportBuilder;

    /**
     * SendEmail constructor.
     * @param \Magento\Framework\App\Helper\Context $context
     * @param \Magento\Store\Model\StoreManagerInterface $storeManager
     * @param \Magento\Framework\Translate\Inline\StateInterface $inlineTranslation
     * @param \Magento\Framework\Mail\Template\TransportBuilder $_transportBuilder
     */
    public function __construct(
        \Magento\Framework\App\Helper\Context $context,
        \Magento\Store\Model\StoreManagerInterface $storeManager,
        \Magento\Framework\Translate\Inline\StateInterface $inlineTranslation,
        \Magento\Framework\Mail\Template\TransportBuilder $_transportBuilder
    )
    {
        $this->_storeManager = $storeManager;
        $this->inlineTranslation = $inlineTranslation;
        $this->_transportBuilder = $_transportBuilder;
        parent::__construct($context);
    }

    /**
     * @param null $store
     * @return bool
     */
    public function isActiveSendEmail($store = null)
    {
        return $this->scopeConfig->isSetFlag(
            self::XML_CONFIG_PATH . 'enabled',
            \Magento\Store\Model\ScopeInterface::SCOPE_STORE,
            $store
        );
    }

    /**
     * @return mixed
     */
    public function getSenderName()
    {
        return $this->scopeConfig->getValue(
            self::XML_CONFIG_PATH . 'sender_name',
            \Magento\Store\Model\ScopeInterface::SCOPE_STORE
        );
    }

    /**
     * @return mixed
     */
    public function getSenderEmail()
    {
        return $this->scopeConfig->getValue(
            self::XML_CONFIG_PATH . 'sender_email',
            \Magento\Store\Model\ScopeInterface::SCOPE_STORE
        );
    }

    /**
     * @return mixed
     */
    public function getTemplate()
    {
        return $this->scopeConfig->getValue(
            self::XML_CONFIG_PATH . 'template',
            \Magento\Store\Model\ScopeInterface::SCOPE_STORE
        );
    }

    /**
     * @return mixed
     */
    public function getCopyTo()
    {
        return $this->scopeConfig->getValue(
            self::XML_CONFIG_PATH . 'copy_to',
            \Magento\Store\Model\ScopeInterface::SCOPE_STORE
        );
    }

    /**
     * @param $customerEmail
     * @param array $emailData
     * @return array
     */
    public function sendBookingConfirmEmail($customerEmail, $emailData = array())
    {
        $result = array();
        if ($this->isActiveSendEmail()) {
            if ($customerEmail) {
                $this->inlineTranslation->suspend();
                try {
                    $emailTemplate = $this->getTemplate();

                    $sender = [
                        'email' => $this->getSenderEmail(),
                        'name' => $this->getSenderName()
                    ];
                    $recipients = explode(",", $customerEmail);

                    $emailCopyTo = $this->getCopyTo();

                    if (count($recipients) == 0) {
                        $result = array(
                            'type' => 'fail',
                            'msg' => __('We can\'t send email right now. Sorry, that\'s all we know.')
                        );
                    } else {
                        foreach ($recipients as $recipient) {
                            $this->_transportBuilder
                                ->setTemplateIdentifier($emailTemplate)
                                ->setTemplateOptions(
                                    [
                                        'area' => \Magento\Framework\App\Area::AREA_FRONTEND,
                                        'store' => $this->_storeManager->getStore()->getId(),
                                    ]
                                )
                                ->setTemplateVars(['data' => $emailData])
                                ->setFrom($sender)
                                ->addTo(trim($recipient));

                            if ($emailCopyTo) {
                                $this->_transportBuilder->addBcc($emailCopyTo);
                            }

                            $transport = $this->_transportBuilder->getTransport();
                            $transport->sendMessage();
                            $this->inlineTranslation->resume();
                        }

                        $result = array(
                            'type' => 'success',
                            'msg' => __('Email has been sent.')
                        );
                    }
                } catch (\Exception $e) {
                    $this->inlineTranslation->resume();
                    $result = array(
                        'type' => 'fail',
                        'msg' => __('We can\'t send email right now. Sorry, that\'s all we know.')
                    );
                }
            } else {
                $result = array(
                    'type' => 'fail',
                    'msg' => __("Please provide customer's email!")
                );
            }
        }
        return $result;
    }
}
```

