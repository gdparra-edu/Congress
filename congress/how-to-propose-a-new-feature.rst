============================
How to propose a new feature
============================

To propose a new feature:

 #. Create a `blueprint <https://blueprints.launchpad.net/congress/+addspec>`__
    describing what your feature does and how it will work. Include just the
    name, title, and summary.

 #. Ask at least one core reviewer to give you feedback. If the feature is
    complex or controversial, the core will ask you to develop a "spec" and add
    it to the congress-specs repo. A spec is a proxy for a blueprint that
    everyone in the community can comment on and help refine. If Launchpad, the
    software OpenStack uses for managing blueprints and bugs, made it possible
    for the community to add comments to a blueprint, we would not need specs
    at all.

 #. Once your blueprint is approved, you may push code that implements that
    blueprint up to Gerrit, including the tag 'Implements-blueprint:
    <blueprint-name>'

To create a spec requested by a core reviewer.

 #. Checkout the congress-specs repo.

 #. Create a new file, whose name is similar to the blueprint name, in the
    current release folder, e.g. congress-specs/specs/kilo/your-spec-here.rst

 #. Use reStructuredText (RST) (an RST tutorial) to describe your new feature
    by filling out the spec template.

 #. Push your changes to the congress-specs repo, a process that is explained
    `here <https://wiki.openstack.org/wiki/Gerrit_Workflow>`__

 #. Go back to your blueprint and add a link (e.g.
    https://github.com/openstack/congress-specs/tree/master/specs/kilo/spec.rst
    ) to your spec in the Specification Location field on the Blueprint details
    .

 #. Participate in the discussion and refinement of your feature via
    `Gerrit reviews <https://review.openstack.org/#/q/status:open+project:
    openstack/congress-specs,n,z>`__

 #. Eventually, core reviewers will either reject or approve your spec

 #. If the spec is approved, a core reviewer will merge it into the repo and
    approve your blueprint.

Note: If your blueprint is either not approved or not implemented during a
release cycle, it must be resubmitted for the next cycle.
