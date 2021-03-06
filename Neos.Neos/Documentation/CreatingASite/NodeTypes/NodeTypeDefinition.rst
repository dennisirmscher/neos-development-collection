.. _node-type-definition:

Node Type Definition
====================

Each Neos ContentRepository Node (we'll just call it Node in the remaining text) has a specific
*node type*. Node Types can be defined in any package by declaring them in
``Configuration/NodeTypes.yaml``.

Each node type can have *one or multiple parent types*. If these are specified,
all properties and settings of the parent types are inherited.

A node type definition can look as follows::

	'My.Package:SpecialHeadline':
	  superTypes:
	    'Neos.Neos:Content': true
	  ui:
	    label: 'Special Headline'
	    group: 'general'
	  properties:
	    headline:
	      type: 'string'
	      defaultValue: 'My Headline Default'
	      ui:
	        inlineEditable: true
	      validation:
	        'Neos.Neos/Validation/StringLengthValidator':
	          minimum: 1
	          maximum: 255

The following options are allowed:

``abstract``
  A boolean flag, marking a node type as *abstract*. Abstract node types can never be used standalone,
  they will never be offered for insertion to the user in the UI, for example.

  Abstract node types are useful when using inheritance and composition, so mark base node types and
  mixins as abstract.

``aggregate``
  A boolean flag, marking a node type as *aggregate*. If a node type is marked as aggregate, it means that:

  - the node type can "live on its own", i.e. can be part of an external URL
  - when moving this node, all node variants are also moved (across all dimensions)
  - Recursive copying only happens *inside* this aggregate, and stops at nested aggregates.

  The most prominent *aggregate* is `Neos.Neos:Document` and everything which inherits from it, like
  `Neos.NodeTypes:Page`.

``superTypes``
  An array of parent node types as keys with a boolean value::

    'Neos.Neos:Document':
      superTypes:
        'Acme.Demo.ExtraMixin': true

    'Neos.Neos:Shortcut':
      superTypes:
        'Acme.Demo.ExtraMixin': false


``constraints``
  Constraint definitions stating which nested child node types are allowed. Also see the dedicated chapter
  :ref:`node-constraints` for detailed explanation::

    constraints:
      nodeTypes:
        # ALLOW text, DISALLOW Image
        'Neos.NodeTypes:Text': true
        'Neos.NodeTypes:Image': false
        # DISALLOW as Fallback (for not-explicitly-listed node types)
        '*': false

``childNodes``
  A list of child nodes that are automatically created if a node of this type is created.
  For each child the ``type`` has to be given. Additionally, for each of these child nodes,
  the ``constraints`` can be specified to override the "global" constraints per type.
  Here is an example::

    childNodes:
      someChild:
        type: 'Neos.Neos:ContentCollection'
        constraints:
          nodeTypes:
            # only allow images in this ContentCollection
            'Neos.NodeTypes:Image': true
            '*': false

  By using ``position``, it is possible to define the order in which child nodes appear in the structure tree.
  An example may look like::

    'Neos.NodeTypes:Page':
      childNodes:
        'someChild':
          type: 'Neos.Neos:ContentCollection'
          position: 'before main'

  This adds a new ContentCollection called someChild to the default page.
  It will be positioned before the main ContentCollection that the default page has.
  The position setting follows the same sorting logic used in Fusion
  (see the :ref:`neos-fusion-reference`).

``label``
  When displaying a node inside the Neos UI (e.g. tree view, link editor, workspace module) the ``label`` option will
  be used to generate a human readable text for a specific node instance (in contrast to the ``ui.label``
  which is used for all nodes of that type).

  The label option accepts an Eel expression that has access to the current node using the ``node`` context variable.
  It is recommended to customize the `label` option for node types that do not yield a sufficient description
  using the default configuration.

  Example::

    'Neos.Demo:Flickr':
      label: ${'Flickr plugin (' + q(node).property('tags') + ')'}

  ``generatorClass``
    Alternatively the class of a node label generator implementing
    ``Neos\ContentRepository\Domain\Model\NodeLabelGeneratorInterface`` can be specified as a nested option.

``options``
  Options for third party-code, the Content-Repository ignores those options but Neos or Packages may use this to adjust
  their behavior.

  ``fusion``
    Options to control the behavior of fusion-for a specific nodeType.

    ``prototypeGenerator``
      The class that is used to generate the default fusion-prototype for this nodeType.

      If this option is set to a className the class has to implement the interface
      ``\Neos\Neos\Domain\Service\DefaultPrototypeGeneratorInterface`` and is used to generate the prototype-code for this node.

      If ``options.fusion.prototypeGenerator`` is set to ``null`` no prototype is created for this type.

      By default Neos has generators for all nodes of type ``Neos.Neos:Node`` and creates protoypes based on
      ``Neos.Fusion:Template``. A template path is assumed based on the package-prefix and the nodetype-name. All properties
      of the node are passed to the template. For the nodeTypes of type ``Neos.Neos:Document``, ``Neos.Neos:Content`` and
      ``Neos.Neos:Plugin`` the corresponding prototype is used as base-prototype.

      Example::

        prototype(Vendor.Site:Content.SpecialNodeType) < prototype(Neos.Fusion:Content) {
          templatePath = 'resource://Vendor.Site/Private/Templates/NodeTypes/Content.SpecialNodeType.html'
          # all properties of the nodeType are passed to the template
          date = ${q(node).property('date')}
          # inline-editable strings additionally get the convertUris processor
          title = ${q(node).property('title')}
          title.@process.convertUris = Neos.Neos:ConvertUris
        }

``ui``
  Configuration options related to the user interface representation of the node type

  ``label``
    The human-readable label of the node type

  ``group``
    Name of the group this content element is grouped into for the 'New Content Element' dialog.
    It can only be created through the user interface if ``group`` is defined and it is valid.

    All valid groups are given in the ``Neos.Neos.nodeTypes.groups`` setting

  ``position``
    Position inside the group this content element is grouped into for the 'New Content Element' dialog.
    Small numbers are sorted on top.

  ``icon``
    This setting defines the icon that the Neos UI will use to display the node type.

    Legacy:
    In Neos versions before 4.0 it was required to use icons from the Fontawesome 3 or 4 versions,
    prefixed with "icon-"

    Current:
    In Neos 4.0, Fontawesome 5 was introduced, enabling the usage of all free Fontawesome icons:
    https://fontawesome.com/icons?d=gallery&m=free
    Those can still be referenced via "icon-[name]", as the UI includes a fallback to the "fas"
    prefix-classes. To be sure which icon will be used, they can also be referenced by their
    icon-classes, e.g. "fas fa-check".


  ``help``
    Configuration of contextual help. Displays a message that is rendered as popover
    when the user clicks the help icon in an insert node dialog.

    ``message``
      Help text for the node type. It supports markdown to format the help text and can
      be translated (see :ref:`translate-nodetypes`).

    ``thumbnail``
      This is shown in the popover and can be supplied in two ways:

      - as an absolute URL to an image (``http://static/acme.com/thumbnails/bar.png``)
      - as a resource URI (``resource://AcmeCom.Website/NodeTypes/Thumbnails/foo.png``)

      If the ``thumbnail`` setting is undefined but an image matching the nodetype name
       is found, it will be used automatically. It will be looked for in
       ``<packageKey>/Resources/Public/NodeTypes/Thumbnails/<nodeTypeName>.png`` with
       ``packageKey`` and ``nodeTypeName`` being extracted from the full nodetype name
       like this:

       ``AcmeCom.Website:FooWithBar`` -> ``AcmeCom.Website`` and ``FooWithBar``

       The image will be downscaled to a width of 342 pixels, so it should either be that
       size to be placed above any further help text (if supplied) or be half that size for
       the help text to flow around it.

  ``inlineEditable``
    If `true`, it is possible to interact with this Node directly in the content view.
    If `false`, an overlay is shown preventing any interaction with the node.
    If not given, checks if any property is marked as ``ui.inlineEditable``.

  ``inspector``
    These settings configure the inspector in the Neos UI for the node type

    ``tabs``
      Defines an inspector tab that can be used to group property groups of the node type

      ``label``
        The human-readable label for this inspector tab

      ``position``
        Position of the inspector tab, small numbers are sorted on top

      ``icon``
        This setting define the icon to use in the Neos UI for the tab

        Currently it's only possible to use a predefined selection of icons, which
        are available in Font Awesome http://fortawesome.github.io/Font-Awesome/3.2.1/icons/.

    ``groups``
      Defines an inspector group that can be used to group properties of the node type

      ``label``
        The human-readable label for this inspector group

      ``position``
        Position of the inspector group, small numbers are sorted on top

      ``icon``
        This setting define the icon to use in the Neos UI for the group

      ``tab``
        The tab the group belongs to. If left empty the group is added to the ``default`` tab.

      ``collapsed``
        If the group should be collapsed by default (true or false). If left empty, the group will be expanded.
  ``creationDialog``
    Creation dialog elements configuration. See :ref:`node-creation-dialog` for more details.
``properties``
  A list of named properties for this node type. For each property the following settings are available.

  .. note:: Your own property names should never start with an underscore ``_`` as that is used for internal
            properties or as an internal prefix.

  ``type``
    Data type of this property. This may be a simple type (like in PHP), a fully qualified PHP class name, or one of
    these three special types: ``DateTime``, ``references``, or ``reference``. Use ``DateTime`` to store dates / time as a
    DateTime object. Use ``reference`` and ``references`` to store references that point to other nodes. ``reference``
    only accepts a single node or node identifier, while ``references`` accepts an array of nodes or node identifiers.

  ``defaultValue``
    Default value of this property. Used at node creation time. Type must match specified 'type'.

  ``ui``
    Configuration options related to the user interface representation of the property

    ``label``
      The human-readable label of the property

    ``help``
      Configuration of contextual help. Displays a message that is rendered as popover
      when the user clicks the help icon in the inspector.

      ``message``
        Help text for this property. It supports markdown to format the help text and can
        be translated (see :ref:`translate-nodetypes`).

    ``reloadIfChanged``
      If `true`, the whole content element needs to be re-rendered on the server side if the value
      changes. This only works for properties which are displayed inside the property inspector,
      i.e. for properties which have a ``group`` set.

    ``reloadPageIfChanged``
      If `true`, the whole page needs to be re-rendered on the server side if the value
      changes. This only works for properties which are displayed inside the property inspector,
      i.e. for properties which have a ``group`` set.

    ``inlineEditable``
      If `true`, this property is inline editable, i.e. edited directly on the page.

    ``aloha``
      Legacy configuration of rich text editor, works for the sake of backwards compatibility, but it
      is advised to use `inline.editorOptions` instead.

    ``inline``
      This section controls the text formatting options the user has available for this property.

      **Note**: When using `inline.editorOptions` anything defined under the legacy `aloha` key for a
      property is ignored. Keep this in mind when using supertypes and mixins.

      Example::

        inline:
          editorOptions:
            placeholder: 'Insert text here'
            autoparagraph: true
            formatting:
              'strong': true
              'em': true
              'u': true
              'sub': true
              'sup': true
              'p': true
              'h1': true
              'h2': true
              'h3': true
              'h4': false
              'h5': false
              'h6': false
              'code': false
              'removeFormat': true
              'table': true
              'a': true
              'ul': true
              'ol': true
              'left': true
              'center': true
              'right': true
              'justify': true

    ``inspector``
      These settings configure the inspector in the Neos UI for the property.

      ``group``
        Identifier of the *inspector group* this property is categorized into in the content editing
        user interface. If none is given, the property is not editable through the property inspector
        of the user interface.

        The value here must reference a groups configured in the ``ui.inspector.groups`` element of the
        node type this property belongs to.

      ``position``
        Position inside the inspector group, small numbers are sorted on top.

      ``hidden``
        If `true`, the editor of this property won't be shown in the inspector group. May be used for :ref:`dynamic-configuration-processing`.

      ``editor``
        Name of the JavaScript Editor Class which is instantiated to edit this element in the inspector.

      ``editorOptions``
        A set of options for the given editor, see the :ref:`property-editor-reference`.

      ``editorListeners``
        Allows to observe changes of other properties in order to react to them. For details see :ref:`depending-properties`

  ``validation``
    A list of validators to use on the property. Below each validator type any options for the validator
    can be given. See below for more information.

.. tip:: Unset a property by setting the property configuration to null (``~``).

Here is one of the standard Neos node types (slightly shortened)::

	'Neos.NodeTypes:Image':
	  superTypes:
	    'Neos.Neos:Content': true
	  ui:
	    label: 'Image'
	    icon: 'icon-picture'
	    inspector:
	      groups:
	        image:
	          label: 'Image'
	          icon: 'icon-image'
	          position: 5
	  properties:
	    image:
	      type: Neos\Media\Domain\Model\ImageInterface
	      ui:
	        label: 'Image'
	        reloadIfChanged: true
	        inspector:
	          group: 'image'
	    alignment:
	      type: string
	      defaultValue: ''
	      ui:
	        label: 'Alignment'
	        reloadIfChanged: true
	        inspector:
	          group: 'image'
	          editor: 'Neos.Neos/Inspector/Editors/SelectBoxEditor'
	          editorOptions:
	            placeholder: 'Default'
	            values:
	              '':
	                label: ''
	              center:
	                label: 'Center'
	              left:
	                label: 'Left'
	              right:
	                label: 'Right'
	    alternativeText:
	      type: string
	      ui:
	        label: 'Alternative text'
	        reloadIfChanged: true
	        inspector:
	          group: 'image'
	      validation:
	        'Neos.Neos/Validation/StringLengthValidator':
	          minimum: 1
	          maximum: 255
	    hasCaption:
	      type: boolean
	      ui:
	        label: 'Enable caption'
	        reloadIfChanged: true
	        inspector:
	          group: 'image'
	    caption:
	      type: string
	      defaultValue: '<p>Enter caption here</p>'
	      ui:
	        inlineEditable: true


