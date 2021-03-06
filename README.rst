Refreshbooks provides a simple synchronous API for manipulating FreshBooks 
invoices, clients, and other data::

    from refreshbooks import api
    
    c = api.OAuthClient(
        'example.freshbooks.com',
        'consumerkey',
        'My Consumer Secret',
        'An existing token',
        'An existing token secret',
        user_agent='Example/1.0'
    )
    
                                          # XML structure inferred from args
    response = c.invoice.create(          # <request method="invoice.create">
        invoice=dict(                     #   <invoice>
            client_id='8',                #     <client_id>8</client_id>
            lines=[                       #     <lines>
                api.types.line(           #       <line>
                    name='Yard Work',     #         <name>Yard Work</name>
                    unit_cost='10',       #         <unit_cost>10</unit_cost>
                    quantity='4'          #         <quantity>4</quantity>
                )                         #       </line>
            ]                             #     </lines>
        )                                 #   </invoice>
    )                                     # </request>
    
    invoice_response = c.invoice.get(     # <request method="invoice.get">
        invoice_id=response.invoice_id    #     <invoice_id>...</invoice_id>
    )                                     # </request>
    
    print "New invoice created: #%s (id %s)" % (
        invoice_response.invoice.number,
        invoice_response.invoice.invoice_id
    )
    
    invoices_response = c.invoice.list() # <request method="invoice.list" />
    
    print "There are %s pages of invoices." % (
        invoices_response.invoices.attrib['pages'],
    )
    
    for invoice in invoices_response.invoices.invoice:
        print "Invoice %s total: %s" % (
            invoice.invoice_id,
            invoice.amount
        )

Consumer keys and secrets can be obtained from FreshBooks. This library
does not handle negotiating for an OAuth token+secret pair; see the
`oauth` module or the OAuth specification for details.

This library also supports the older token-based API authorization 
scheme::

    c = api.TokenClient(
        'example.freshbooks.com',
        'My API token',
        user_agent='Example/1.0'
    )
    
    # ... as above ...

API methods return lxml.objectify.ObjectifiedDataElement trees, which
can be manipulated as Python objects with the same structure as the 
underlying XML.

If you are having trouble accessing items as in:

    items_response = c.items.list()
    for item in items_response.items.item:
        print item.item_id

Adjust your syntax to use dictionary item lookup:

    items_response = c.items.list()
    for item in items_response['items'].item:
        print item.item_id

ObjectifiedDataElement provides a method named items which shadows the 
items element in the response. Accessing items with dictionary lookup 
syntax is the known work-around.

To run tests:

    python setup.py nosetests

To run network-accessing integration tests against httpstat.us:

    python setup.py nosetests --attr=integration

References:

 - http://developers.freshbooks.com/ - The FreshBooks API
 - http://developers.freshbooks.com/authentication-2/#OAuth - FreshBooks and OAuth
