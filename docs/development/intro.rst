Development
===========

Introduction
::::::::::::

Bungeni Source Code Style Guide
-------------------------------

Recommendations for Bungeni source code style, with the aim of enhancing source code read-ability, grep-ability and print-ability and consequently, understand-ability, debug-ability, test-ability, maintain-ability, etc.

However, if conforming to these source style recommendations reduces any of the above code qualities in any way then conformance should not be adhered to -- use your own good judgement instead.

All Formats
:::::::::::

Guidelines that apply to all project source code, irrespective of format e.g. .py, .xml, .zcml, .js, .css, etc, even if focus here is primarily on python and XML.

Only use editing tools that do not automatically modify source code format in any way! This means you may not use any tool that does any kind of source reformatting e.g. you may not use tools like oXygen XML Editor that may auto-reformat whitespace or even change the order of an XML element's attributes. In general, what you want to have is a good text editor that respects your text source.

Line length, indentation, continuation lines
Limit all lines to a maximum of 79 characters.
Use 4 spaces per indentation and 1 indentation per logical depth level.
Continuation lines should be indented as per their nesting level i.e. 4 spaces per depth level implied by an open parenthesis, brace, bracket or any other way.
The closing thingy (e.g. parenthesis, brace, bracket, tag) of a continuation line should be directly below the beginning of the opening symbol.
Only use multiple lines when a single line (of max 79 chars) is not sufficient.
Always specify the most key of parameters on the opening line (for long collection-like code segments).
Quote characters
Prefer the double-quote " instead of the single-quote ' character, whenever you have the choice.

Consistency across languages, order of attributes/parameters
It is very common for a project to make use of more than one language, and in many cases there are clear references to constructs of one language from an other e.g. an XML definition of a python construct.

In some languages the order or parameters is (sometimes) relevant e.g. in python, but in other languages such an order is not relevant e.g. attributes in XML. Whenever an equivalent list of attributes or parameters are present (be it as a list of XML attributes, or as the signature of a python callable) then the order should be consistent between the two languages.

For the cases when there is such an order to respect, the order should be clearly commented in appropriate places in the source itself.

Revisit Comments
Use Revisit comments for code that is temporary, a short-term solution to be reworked, or for code on which there is still questions to be resolved or understood. Multiple related Revisit comments (that use same label) may be used across the project source, and across different languages.

Note that authoring a revisit comment is not a commitment to revisit it yourself.

A Revisit comments should have a:

!+ : must explicitly start with these 2 characters
label : for easier grepping, and to be able to relate multiple comments across the project source (even in different languages)
author identifier : so others know who to follow-up with if needed
date : so it is easier to judge relevance and status at a later time
description : should also indicate what conditions/events would render the comment obsolete.
These pieces of information should be formatted in the following way:
    !+LABEL(author, date) description...
and should be specified in a comment as per the source format i.e. any of python, XML, doctest text files, HTML templates, css, js, etc.

As example, here are 2 related revisit comments in different files:

models/interfaces.py

''' !+DATERANGEFILTER(mr, dec-2010) disabled until intention is understood
class IDateRangeFilter(interface.Interface):
    """Adapts a model container instance and a SQLAlchemy query
    object, applies a date range filter and returns a query.

    Parameters: ``start_date``, ``end_date``.

    These must be bound before the query is executed.
    """
'''
models/configure.zcml

    <!-- !+DATERANGEFILTER(mr, dec-2010)
    <adapter for=".domain.Bill"
        provides=".interfaces.IDateRangeFilter"
        factory=".daterange.bill_filter"
    />
    ...
    -->
Naming
Permissions
All permission names should have the form:

    bungeni.{object[ .subobject ]}.{Action}
where :

object is a noun indicating the target type of object (all lowercase, may optionally have additional dot-separated subtypes).
Action is a verb indicating the permitted target action (capitalized).
Python
In addition to style guidelines for All Formats above, all Bungeni python source code should also follow additional python-specific style rules.

For any styling issue not explicitly specified here, heed the recommendations of the following Python Style Guides:

PEP 8 - Style Guide for Python Code in particular the guidelines concerning blank lines, whitespace, comments, naming conventions and programming recommendations.

Regarding this one you may find the following tools useful:
pep8 checker tool
PythonTidy
Google Python Style Guide

In particular, the recommendations pertaining to:
Imports: for packages and modules only.
Packages: import each module using the full pathname location of the module.
Nested/Local/Inner Classes and Functions: nested/local/inner classes and functions are fine.
Comments: use the right style for module, function, method and in-line comments.
Whitespace
Exactly as specified in PEP 8, but following cases are not explicitly covered there:

Just as for other operators, there should be a space surrounding the % string formatting operator:

        assert len(states), "Workflow [%s] defines no states" % (self.name)
and not:

        assert len(states), "Workflow [%s] defines no states"%(self.name)
Contrary to the formatting of brackets and parenthesis for literal lists and tuples, there should be a space after the opening and before the closing of list comprehensions and tuple comprehensions (generators):

    set([ (p[1], p[2]) for p in states[0].permissions ])
    ( (role, s) for s, p, role in self.permissions if p == permission )
and not:

    set([(p[1], p[2]) for p in states[0].permissions])
    ((role, s) for s, p, role in self.permissions if p == permission)
Quote characters
Always use a single double-quote char " to delineate a short literal string. Only use single-quote ' when the literal string itself contains double-quote characters that would otherwise require explicit escaping.

Always use triple double-quote chars """ to delineate a multi-line string. Only use triple single-quotes ''' when the literal string contains itself triple double-quote characters that would otherwise require explicit escaping.

The use of triple single-quotes ''' should be reserved only to temporarily comment out entire code sections.

For long string that should not include any additional whitespace as a result of pretty source formatting should use the auto-concatenation feature of adjacent strings e.g. (preferably):

    long_string = "some initial text as part of a single long line string " \
        "and some more text for the same string and some more text for the " \
        "same string"
or:

    long_string = ("some initial text as part of a single long line string "
        "and some more text for the same string and some more text for the "
        "same string")
but not:

    long_string = ("some initial text as part of a single long line string " +
        "and some more text for the same string and some more text for the " +
        "same string")
Quoting inlined source code strings
Always delineate embedded code with triple double-quote chars """ -- even if the string is not multi-line. Example, here's a snippet of inlined SQL:

        rdb.CheckConstraint("""active_p in ('A', 'I', 'D')"""),
Multi-line parameter lists, literal lists and objects
The exact same Line length, indentation, continuation lines style rules for all formats, above, are the rules to apply here, with the one essential rule to always remember being one indentation level per logical nesting level.

Some examples:

    columns = [
        rdb.Column("version_id", rdb.Integer, primary_key=True),
        rdb.Column("content_id", rdb.Integer, rdb.ForeignKey(table.c[fk_id])),
        rdb.Column("change_id", rdb.Integer,
            rdb.ForeignKey("%s_changes.change_id" % entity_name)
        ),
        rdb.Column("manual", rdb.Boolean, nullable=False, default=False),
    ]
mapper(domain.User, schema.users,
    properties={"user_addresses": relation(domain.UserAddress)}
)
mapper(domain.QuestionVersion, schema.question_versions,
    properties={
        "change": relation(domain.QuestionChange, uselist=False),
        "head": relation(domain.Question, uselist=False),
        "attached_files": relation(domain.AttachedFileVersion,
            primaryjoin=rdb.and_(
                schema.question_versions.c.content_id ==
                    schema.attached_file_versions.c.item_id,
                schema.question_versions.c.version_id ==
                    schema.attached_file_versions.c.file_version_id
            ),
            foreign_keys=[schema.attached_file_versions.c.item_id,
                schema.attached_file_versions.c.file_version_id
            ]
        ),
    }
)
Same rules apply for multi-line block-opening statements, giving the following results (with block opener implying 1 nesting level and the parameter continuation lines implying another):

    def __init__(self,
            # resolvable python callable
            condition="owner_receive_notification",
            # i18n, template source
            subject="${item.class_name} ${item.status}: ${item.short_name}",
            # template source
            from_="${site.clerk_email}",
            # template source
            to="${item.owner_email}",
            # i18n, template source
            body="${item.class_name} ${item.status}: ${item.short_name}",
            # documentational note
            note=None
        ):
        self.condition = capi.get_workflow_condition(condition)
        ...
    if (IAlchemistContent.providedBy(context) and
            IDCDescriptiveProperties.providedBy(context) and
            ILocation.providedBy(context)
        ):
        title = context.title
Multiple forms for same -- adopt and use one
Comparison operators
Always use != and not <> (that is equivalent).

XML / ZCML
Blank lines
If you must, use blank line sparingly -- only use a single blank line between elements on the infrequent occasion when those elements are completely unrelated to each other.

Multi-line elements
Again, the same Line length, indentation, continuation lines style rules for all formats, above, are the rules to apply here.

Non-empty XML elements should have the closing tag:

either on same single line as the opening tag
or at exactly same indent as opening tag
Same goes for the /> ending of empty XML elements.

There should not be any whitespace preceeding the closing > of a tag; but, to enhance the distinction between empty and non-empty elements, there should be a single space preceeding the /> ending of empty XML elements.

Examples:

    <class class=".Ministry"><require like_class=".Group" /></class>
    <class class="bungeni.models.domain.Ministry">
        <require like_class="bungeni.models.domain.Group" />
    </class>
    <browser:menu id="admin_navigation" title="Actions Admin" />
    <browser:menuItem menu="admin_navigation"
        for="bungeni.models.interfaces.IBungeniApplication"
        title="Application Settings"
        action="settings"
        layer=".interfaces.IAdministratorWorkspace"
        permission="zope.ManageSite"
    />
When an element has a clear key attribute -- i.e. the one most obvious attribute by which the element could be identified or grouped e.g. the id or name attribute, the menu attribute for <menuItem> elements, etc. -- it should be specified first and on same line as opening tag. For example:

    <adapter factory=".workspace.WorkspaceContainerTraverser"
        permission="zope.Public"
        trusted="true"
    />