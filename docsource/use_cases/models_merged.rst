Models Merged
+++++++++++++

From version 12.0 to version 13.0, in the module ``survey``, the models
``survey.page`` has been merged into ``survey.question``.

Analysis
--------

.. code-block:: text

    ---Models in module 'survey'---
    obsolete model survey.page (renamed to survey.question)

    ---Fields in module 'survey'---
    survey       / survey.page              / description (html)            : DEL
    survey       / survey.page              / question_ids (one2many)       : DEL relation: survey.question
    survey       / survey.page              / sequence (integer)            : DEL
    survey       / survey.page              / survey_id (many2one)          : DEL relation: survey.survey, required
    survey       / survey.page              / title (char)                  : DEL required

See `Full V13 Analysis File <https://github.com/OCA/OpenUpgrade/blob/13.0/addons/survey/migrations/13.0.3.0/openupgrade_analysis.txt#L13-L17>`_.

Source Code Differences
-----------------------

Version 12.0
""""""""""""



.. code-block:: python

    class SurveyPage(models.Model):
        _name = "survey.page"

        title = fields.Char(string="Page Title")
        description = fields.Html(string="Description")
        sequence = fields.Integer(string="Page number")
        survey_id = fields.Many2one(comodel_name="survey.survey", string='Survey')
        question_ids = fields.One2many(comodel_name="survey.question", inverse_name="page_id")


    class SurveyQuestion(models.Model):
        _name = "survey.question"

        description = fields.Html(string="Description")
        sequence = fields.Integer("Sequence")
        survey_id = fields.Many2one(comodel_name="survey.survey", related="page_id.survey_id")
        page_id = fields.Many2one(comodel_name="survey.page", string="Survey page")
        question = fields.Char(string="Question Name")

See `Full V12 Code Source <https://github.com/odoo/odoo/blob/12.0/addons/survey/models/survey.py>`_.


Version 13.0
""""""""""""

.. code-block:: python

    class MailActivityType(models.Model):

        _name = 'mail.activity.type'

        default_note = fields.Html(
            string="Default Description",
            translate=True,
        )

See `Full V15 Code Source <https://github.com/odoo/odoo/blob/13.0/addons/survey/models/survey_question.py>`_.

Result without migration script / Expected Result
-------------------------------------------------

V14 table ``mail_activity_type``
""""""""""""""""""""""""""""""""

.. csv-table::
   :header: "id", "name", "default_description"

   "1", "Email", "<p>A description</p>"
   "2", "Call", ""
   "3", "Meeting", "<p>Another description</p>"

V15 table mail_activity_type (Without migration script)
"""""""""""""""""""""""""""""""""""""""""""""""""""""""

.. csv-table::
   :header: "id", "name", "default_note"

   "1", "Email", ""
   "2", "Call", ""
   "3", "Meeting", ""

**Problem** : the data is lost during them migration process, and the new column is empty.

V15 table mail_activity_type (With migration script)
""""""""""""""""""""""""""""""""""""""""""""""""""""

.. csv-table::
   :header: "id", "name", "default_note"

   "1", "Email", "<p>A description</p>"
   "2", "Call", ""
   "3", "Meeting", "<p>Another description</p>"

Contribution to OpenUpgrade
---------------------------

Update ``upgrade_analysis_work.txt`` file
"""""""""""""""""""""""""""""""""""""""""

* Mention the operation performed, starting with ``# DONE:``

.. code-block:: text

    ---Models in module 'survey'---
    obsolete model survey.page (renamed to survey.question)
    # DONE: post-migration: merged into survey.question


See `Full V15 Work Analysis File <https://github.com/OCA/OpenUpgrade/blob/13.0/addons/survey/migrations/13.0.3.0/openupgrade_analysis_work.txt#L6-L7>`_.

Write migration Script
""""""""""""""""""""""

in the ``pre-migration.py`` script add:

.. code-block:: python

    from openupgradelib import openupgrade

    def _rename_fields(env):
        openupgrade.rename_fields(
            env,
            [
                (
                    "mail.activity.type",
                    "mail_activity_type",
                    "default_description",
                    "default_note",
                ),
            ]
        )

    @openupgrade.migrate()
    def migrate(env, version):
        _rename_fields(env)

See `Full pre migration Script <https://github.com/OCA/OpenUpgrade/blob/15.0/openupgrade_scripts/scripts/mail/15.0.1.5/pre-migration.py>`_.
