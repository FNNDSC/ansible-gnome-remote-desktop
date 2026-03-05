# `fnndsc.gnome_remote_desktop`

An [Ansible Role](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
for setting up `gnome-remote-desktop` on Ubuntu 24.04.

## Motivation

This sets up `gnome-remote-desktop` in "system-level" mode, meaning it supports
**multi-user** and **headless** use cases. `gnome-remote-desktop` is somewhat
well "integrated" e.g. it supports Wayland and user login via GDM. How it works
is that when a user connects, they see the GDM login screen. After logging in,
the user arrives to a transient session with `gnome-shell`. The session is
closed when the user exits their RDP client (i.e. you cannot stay logged in).

### Alternative Solutions

You do _not_ need this role if you want a single-user remote desktop where
physical access is convenient. That is much easier to set up, simply toggle
the option in "Settings" (a.k.a. `gnome-control-center`).

The prior art to `gnome-remote-desktop` include [TurboVNC](https://turbovnc.org/)
and [TigerVNC](https://tigervnc.org/). Those are not integrated with GNOME,
hence they are more complicated to set up while also being more flexible.
There is also [xrdp](https://www.xrdp.org/) (which does not support Wayland)
and companies offering commercial solutions such as [RustDesk](https://rustdesk.com/).

## Role Variables

You must run a playbook with this role at least once with the two variables
`gnome_remote_desktop_rdp_username` and `gnome_remote_desktop_rdp_password`
defined. These set the RDP credentials, which are required to make the RDP
connection (these should be different from your user account credentials).

## Example Playbook

```yaml
---
- hosts: all
  vars_prompt:
    - name: gnome_remote_desktop_rdp_username
      prompt: RDP username
      private: false
    - name: gnome_remote_desktop_rdp_username
      prompt: RDP password
      private: true
  roles:
    - fnndsc.gnome_remote_desktop
```

## Troubleshooting

Try restarting _both_ GDM and gnome-remote-desktop:

```ShellSession
systemctl restart gdm3.service gnome-remote-desktop.service
```

Or straight up reboot.

## Development Notes

- The right way to configure system-level RDP is by using `grdctl --system rdp`
  subcommands, but here we implement the equivalent effects by directly
  creating files with
  [`ansible.builtin.template`](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/template_module.html)
- When a user enables RDP for just themselves in GNOME Settings, `gnome-control-center`
  generates the TLS certificates for them ([source](https://gitlab.gnome.org/GNOME/gnome-control-center/-/blob/5c7f48eccc1df6e798194e6f69f05a95c980ca6b/panels/system/remote-desktop/cc-desktop-sharing-page.c#L203)).
  However when setting up system-level RDP using `grdctl --system rdp enable`,
  these certificates must be generated manually using `winpr-makecert`
- When system-level RDP is enabled in GNOME Settings version 48, it creates the certificates
  at `/var/lib/gnome-remote-desktop/.local/share/gnome-remote-desktop/certificates/rdp-tls.{crt,key}`
  with `subject=/CN=GNOME/C=US`
