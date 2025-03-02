`core:AttributeLimit`
=====================

A filter that limits the attributes (and their values) sent to a service provider.

If the configuration is empty, the filter will use the attributes configured in the `attributes` option in the SP
metadata. The configuration is a list of attributes that should be allowed. In case you want to limit an attribute to
release some specific values, make the name of the attribute the key of the array, and its value an array with all the
different values allowed for it.

Examples
--------

Here you will find a few examples on how to use this simple module:

Limit to the `cn` and `mail` attribute:

    'authproc' => [
        50 => [
            'class' => 'core:AttributeLimit',
            'cn', 'mail'
        ],
    ],

Allow `eduPersonTargetedID` and `eduPersonAffiliation` by default, but allow the metadata to override the limitation.

    'authproc' => [
        50 => [
            'class' => 'core:AttributeLimit',
            'default' => true,
            'eduPersonTargetedID', 'eduPersonAffiliation',
        ],
    ],

Only allow specific values for an attribute.

    'authproc' => [
        50 => [
            'class' => 'core:AttributeLimit',
            'eduPersonEntitlement' => ['urn:mace:surf.nl:surfdrive:quota:100']
        ],
    ],

Only allow specific values for an attribute ignoring case.

    'authproc' => [
        50 => [
            'class' => 'core:AttributeLimit',
            'eduPersonEntitlement' => [
                'ignoreCase' => true,
                'URN:mace:surf.nl:SURFDRIVE:quota:100'
             ]
        ],
    ],

Only allow specific values for an attribute that match a regex pattern

    'authproc' => [
        50 => [
            'class' => 'core:AttributeLimit',
            'eduPersonEntitlement' => [
                'regex' => true,
                '/^urn:mace:surf/',
                '/^urn:x-IGNORE_Case/i',
            ]
        ],
    ],

Don't allow any attributes by default, but allow the metadata to override it.

    'authproc' => [
        50 => [
            'class' => 'core:AttributeLimit',
            'default' => true,
        ],
    ],

In order to just use the list of attributes defined in the metadata for each service provider, configure the module
like this:

    'authproc' => [
        50 => 'core:AttributeLimit',
    ],

Then, add the allowed attributes to each service provider metadata, in the `attributes` option:

    $metadata['https://saml2sp.example.org'] = [
        'AssertionConsumerService' => 'https://saml2sp.example.org/simplesaml/module.php/saml/sp/assertionConsumerService/default-sp',
        'SingleLogoutService' => 'https://saml2sp.example.org/simplesaml/module.php/saml/sp/singleLogoutService/default-sp',
        ...
        'attributes' => ['cn', 'mail'],
        ...
    ];

Now, let's look to a couple of examples on how to filter out attribute values. First, allow only the entitlements known
to be used by a service provider (among other attributes):

    $metadata['https://saml2sp.example.org'] = [
        'AssertionConsumerService' => 'https://saml2sp.example.org/simplesaml/module.php/saml/sp/assertionConsumerService/default-sp',
        'SingleLogoutService' => 'https://saml2sp.example.org/simplesaml/module.php/saml/sp/singleLogoutService/default-sp',
        ...
        'attributes' => [
            'uid',
            'mail',
            'eduPersonEntitlement' => [
                'urn:mace:example.org:admin',
                'urn:mace:example.org:user',
            ],
        ],
        ...
    ];

Now, an example on how to normalize the affiliations sent from an identity provider, to make sure that no custom
values ever reach the service providers. Bear in mind that this configuration can be overridden by metadata:

    'authproc' => [
        50 => 'core:AttributeLimit',
        'default' => true,
        'eduPersonAffiliation' => [
            'student',
            'staff',
            'member',
            'faculty',
            'employee',
            'affiliate',
        ],
    ],
