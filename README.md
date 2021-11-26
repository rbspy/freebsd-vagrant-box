# freebsd-vagrant-box

This repo defines a Vagrant box for building rbspy on FreeBSD.

To test rbspy, we need to compile and install multiple versions of Ruby, and we need to install a Rust toolchain. This is a slow process that we don't want to run during every CI job. The purpose of this box is to provide a functional base system for rbspy's CI jobs (and for one-off troubleshooting) so that nothing needs to be installed. Just boot the box and start using `cargo`.

To download the box, see the [releases](https://github.com/rbspy/freebsd-vagrant-box/releases) page.

## Making a new version

1. Make any desired changes to `Vagrantfile` or the GitHub Actions workflow.
2. Commit the changes, and create a new tag based on today's date using the format `YYYYMMDD`. Push the commit and tag.
3. When the build finishes, it will automatically create a new release with the `.box` file attached. You can copy the link to this asset and download it to your computer, from your CI workflows, etc. See [rbspy's CI workflow](https://github.com/rbspy/rbspy/blob/2387d8dafb7f3b7d210800d78f69cbd808b415fc/.github/workflows/ci.yml#L196-L203) for an example.

## Notes and caveats

- In order to attach the box to a GitHub release, it must be less than 2GB in size.
- It's tempting to use Vagrant's shared folder feature, but I've found it to be very flaky when using VirtualBox with FreeBSD guests. Files aren't immediately visible, and sometimes filesystem operations will fail for unclear reasons. This causes lots of issues with Cargo builds. Maybe future versions of these tools will work better.
- It would be nice to find a slimmer, more minimal base box to use. We don't need a lot of the packages that FreeBSD includes by default.