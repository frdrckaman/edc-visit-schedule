|pypi| |travis| |codecov| |downloads|

edc-visit-schedule
------------------

Add data collection schedules to your app.

Installation
============

Add to settings:

.. code-block:: python
    
    INSTALLED_APPS = [
        ...
        'edc_visit_schedule.apps.AppConfig',
        ...
    ]

Overview
========

* A ``Visit Schedule`` lives in your app in ``visit_schedules.py``. Each app can declare and register one or more visit schedules in its ``visit_schedules`` module. Visit schedules are loaded when ``autodiscover`` is called from ``AppConfig``.
* A ``VisitSchedule`` contains ``Schedules`` which contain ``Visits`` which contain ``Crfs`` and ``Requisitions``.
* A ``schedule`` is effectively a "data collection schedule" where each contained ``visit`` represents a data collection timepoint.
* A subject is put on a ``schedule`` by the schedule's ``onschedule`` model and taken off by the schedule's ``offschedule`` model. In the example below we use models ``OnSchedule`` and ``OffSchedule`` to do this for schedule ``schedule1``.

Usage
=====

First, create a file ``visit_schedules.py`` in the root of your app where the visit schedule code below will live.


Next, declare lists of data ``Crfs`` and laboratory ``Requisitions`` to be completed during each visit. For simplicity, we assume that every visit has the same data collection requirement (not usually the case).

.. code-block:: python
    
    from myapp.models import SubjectVisit, OnSchedule, OffSchedule, SubjectDeathReport, SubjectOffstudy

    from edc_visit_schedule.site_visit_schedules import site_visit_schedules
    from edc_visit_schedule.schedule import Schedule
    from edc_visit_schedule.visit import Crf, Requisition, FormsCollection
    from edc_visit_schedule.visit_schedule import VisitSchedule
    
    
    crfs = FormsCollection(
        Crf(show_order=10, model='myapp.crfone'),
        Crf(show_order=20, model='myapp.crftwo'),
        Crf(show_order=30, model='myapp.crfthree'),
        Crf(show_order=40, model='myapp.crffour'),
        Crf(show_order=50, model='myapp.crffive'),
    )
    
    requisitions = FormsCollection(
        Requisition(
            show_order=10, model='myapp.subjectrequisition', panel_name='Research Blood Draw'),
        Requisition(
            show_order=20, model='myapp.subjectrequisition', panel_name='Viral Load'),
    )

Create a new visit schedule:

.. code-block:: python
    
    subject_visit_schedule = VisitSchedule(
        name='subject_visit_schedule',
        verbose_name='My Visit Schedule',
        death_report_model=SubjectDeathReport,
        offstudy_model=SubjectOffstudy,
        visit_model=SubjectVisit)


Visit schedules contain ``Schedules`` so create a schedule:

.. code-block:: python
    
    schedule = Schedule(
        name='schedule1',
        onschedule_model='myapp.onschedule',
        offschedule_model='myapp.offschedule')

Schedules contains visits, so decalre some visits and add to the ``schedule``:

.. code-block:: python
    
    visit0 = Visit(
        code='1000',
        title='Visit 1000',
        timepoint=0,
        rbase=relativedelta(days=0),
        requisitions=requisitions,
        crfs=crfs)

    visit1 = Visit(
        code='2000',
        title='Visit 2000',
        timepoint=1,
        rbase=relativedelta(days=28),
        requisitions=requisitions,
        crfs=crfs)

    schedule.add_visit(visit=visit0)
    schedule.add_visit(visit=visit1)


Add the schedule to your visit schedule:

.. code-block:: python
    
    schedule = subject_visit_schedule.add_schedule(schedule)

Register the visit schedule with the site registry:

.. code-block:: python
    
    site_visit_schedules.register(subject_visit_schedule)

When Django loads, the visit schedule class will be available in the global ``site_visit_schedules``.

The ``site_visit_schedules`` has a number of methods to help query the visit schedule and some related data.

 **Note:** The ``schedule`` above was declared with ``onschedule_model=OnSchedule``. An on-schedule model uses the ``CreateAppointmentsMixin`` from ``edc_appointment``. On ``onschedule.save()`` the method ``onschedule.create_appointments`` is called. This method uses the visit schedule information to create the appointments as per the visit data in the schedule. See also ``edc_appointment``.

OnSchedule and OffSchedule models
=================================

Two models_mixins are available for the the on-schedule and off-schedule models, ``OnScheduleModelMixin`` and ``OffScheduleModelMixin``. OnSchedule/OffSchedule models are specific to a ``schedule``. The ``visit_schedule_name`` and ``schedule_name`` are declared on the model's ``Meta`` class attribute ``visit_schedule_name``.

For example:

.. code-block:: python
    
    class OnSchedule(OnScheduleModelMixin, CreateAppointmentsMixin, RequiresConsentModelMixin, BaseUuidModel):
        
        class Meta(EnrollmentModelMixin.Meta):
            visit_schedule_name = 'subject_visit_schedule.schedule1'
            consent_model = 'myapp.subjectconsent'
    
    
    class OffSchedule(OffScheduleModelMixin, RequiresConsentModelMixin, BaseUuidModel):
    
        class Meta(OffScheduleModelMixin.Meta):
            visit_schedule_name = 'subject_visit_schedule.schedule1'
            consent_model = 'myapp.subjectconsent'


.. |pypi| image:: https://img.shields.io/pypi/v/edc-visit-schedule.svg
    :target: https://pypi.python.org/pypi/edc-visit-schedule
    
.. |travis| image:: https://travis-ci.org/clinicedc/edc-visit-schedule.svg?branch=develop
    :target: https://travis-ci.org/clinicedc/edc-visit-schedule
    
.. |codecov| image:: https://codecov.io/gh/clinicedc/edc-visit-schedule/branch/develop/graph/badge.svg
  :target: https://codecov.io/gh/clinicedc/edc-visit-schedule

.. |downloads| image:: https://pepy.tech/badge/edc-visit-schedule
   :target: https://pepy.tech/project/edc-visit-schedule
