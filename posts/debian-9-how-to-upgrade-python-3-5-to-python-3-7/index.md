# How to Upgrade Python on Debian, Ubuntu and Kali Linux (and Set the Default Version Safely)

> Install a newer Python on Debian, Ubuntu or Kali without breaking apt: python-is-python3, uv, the deadsnakes PPA, or make altinstall from source. Tested October 2026.

Source: https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-19
Updated: 2026-10-08
Tags: python, debian, ubuntu, kali-linux, linux, uv

> **Updated October 2026:** this page replaces six separate posts I wrote between 2019 and 2021 about upgrading Python on Debian 9 and 10, Ubuntu 19.10 and Kali, and switching the `python` command from 2.7 to 3. The old URLs redirect here. Everything in the first half was re-tested on 8 October 2026 in fresh Debian 12 and Ubuntu 22.04 containers. The original 2019 from-source walkthrough is kept at the end because it still works, with one important correction.

## Quick answer

Pick the row that matches what you're trying to do. The one rule behind all of them: never replace the system `python3` that apt installed. Debian and Ubuntu tooling (apt itself, `lsb_release`, cloud-init, lots of system scripts) is written against that exact interpreter, and pointing it somewhere else is how you end up with a box that can't update itself.

| You want | Do this |
|---|---|
| `python` says "command not found" | `sudo apt install python-is-python3` |
| A newer Python for one project | `uv python install 3.13` then `uv venv --python 3.13` |
| A newer Python system-wide on Ubuntu | deadsnakes PPA, `sudo apt install python3.12 python3.12-venv` |
| A newer Python system-wide on Debian | build from source with `make altinstall` |
| To switch what `python` points at on an old 2.7 box | `update-alternatives` (end of this post) |

## Check what you have

```bash
python3 --version
ls -l /usr/bin/python*
```

On a fresh Debian 12 that's Python 3.11.2. Ubuntu 22.04 ships 3.10, 24.04 ships 3.12. Kali tracks Debian testing, so it's whatever Debian testing has that week. On all of them the interpreter is `python3`. Whether a bare `python` command exists at all is a separate question, which is the next section.

## Make the `python` command work

If scripts or tutorials call `python` and you get "command not found", you don't need another interpreter. You need the one-line package that adds the symlink:

```bash
sudo apt install python-is-python3
python --version    # now the same as python3 --version
```

That's the whole fix on Debian 12, Ubuntu 20.04 and newer, and Kali. It's what the old `update-alternatives` dance at the bottom of this post was trying to achieve by hand.

## A newer Python for a project: uv

For most of what I do now, I don't change the system Python at all. [uv](https://docs.astral.sh/uv/) downloads a standalone build of whichever version I ask for and keeps it out of the way of apt:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv python install 3.13
uv venv --python 3.13 .venv
source .venv/bin/activate
python --version    # Python 3.13.x
```

Tested on Debian 12: `uv python install 3.13` pulled 3.13.16 in a few seconds and `uv run --python 3.13 python --version` reported it. pyenv does the same job if you'd rather compile. Either way the system interpreter is untouched and each project pins its own.

## A newer Python system-wide on Ubuntu: deadsnakes

Ubuntu has a well-maintained PPA that packages every current Python properly, so you get `python3.12` alongside the system `python3` with no compiling:

```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.12 python3.12-venv
python3.12 --version
python3.12 -m venv ~/venv312
```

Tested on Ubuntu 22.04: `python3.12 --version` gave 3.12.15 while `python3 --version` still reported the system 3.10.12, which is exactly what you want. Note `python3.12` is a separate command. Don't alias `python3` to it.

## A newer Python system-wide on Debian: build from source

Debian doesn't have a deadsnakes equivalent, so if you need a system-wide newer interpreter the honest answer is still to compile it. The install step is where my 2019 post was wrong: use `make altinstall`, not `make install`. `altinstall` installs `python3.13` and leaves `python3` alone. `install` overwrites the `python3` symlink and that's the apt-breaking mistake from the first section.

```bash
sudo apt install build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
  libsqlite3-dev libffi-dev liblzma-dev wget
cd /tmp
wget https://www.python.org/ftp/python/3.13.7/Python-3.13.7.tar.xz
tar xf Python-3.13.7.tar.xz
cd Python-3.13.7
./configure --enable-optimizations
make -j"$(nproc)"
sudo make altinstall
python3.13 --version
```

The dependency list and `./configure` were checked on Debian 12 on 8 October 2026. Pick the current release from [python.org/ftp/python](https://www.python.org/ftp/python/). `make` takes a while with `--enable-optimizations`, which is the trade for a faster interpreter. If the box is small, drop that flag.

## Kali Linux

Kali is Debian testing underneath, so every method above applies as written. Kali 2020.1 was the release that moved to Python 3 by default, and the Kali-specific post I wrote back then was only ever the `update-alternatives` trick below. I haven't re-tested on a full Kali install this time round (the minimal container image doesn't ship Python at all), so treat the Debian results as the reference.

## Python 2.7

Python 2.7 reached end of life on 1 January 2020 and the final release, 2.7.18, came in April 2020. Nothing current ships it. If you've inherited a 2.7 project, the official [porting guide](https://docs.python.org/3/howto/pyporting.html) is still the place to start, and `python-is-python2` exists on Debian for the brief period you need both side by side.

## Troubleshooting

**`apt` or `apt-get` throws Python errors after an upgrade**
You replaced the system `python3`. Restore the symlink to the distro version (`ls /usr/bin/python3.*` shows which one) with `sudo ln -sf /usr/bin/python3.11 /usr/bin/python3`, substituting your distro's version.

**`lsb_release` fails with ImportError**
Same cause, a non-system interpreter answering to `python3`. Fix as above. The 2019 workaround of symlinking `lsb_release.py` into site-packages papered over this.

**`pip` installs into the wrong Python**
Always call pip through the interpreter you mean: `python3.12 -m pip install ...`, or better, work inside a venv.

**`add-apt-repository: command not found`**
Install `software-properties-common` first.

## The original 2019 method (Debian 9, Python 3.5 to 3.7)

Kept for reference. This is what the page said from February 2019, lightly tidied. The one correction, as above: use `make altinstall`.

**Check your version**

```bash
python3 -V
```

**Download and extract**

```bash
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tar.xz
tar xf Python-3.7.3.tar.xz
cd ./Python-3.7.3
```

**Make and install**

```bash
./configure
make
make altinstall    # was "make install" in the original, which clobbers python3
```

**Switching the `python` command with update-alternatives**

On the 2019-era systems where `python` still meant 2.7, this registered both interpreters and gave 3.x the higher priority:

```bash
update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
update-alternatives --install /usr/bin/python python /usr/bin/python3.7 2
update-alternatives --config python
```

`update-alternatives --config python` also lets you flip between them later. On anything current, `python-is-python3` does this in one package.

**Fixing pip and lsb_release afterwards**

```bash
ln -s /usr/share/pyshared/lsb_release.py /usr/local/lib/python3.7/site-packages/lsb_release.py
pip3 install --upgrade pip
```

That symlink was a symptom of having replaced `python3`. With `altinstall` it isn't needed.

---

Six posts, one page, and the advice is shorter than it used to be: don't touch the system interpreter, let uv or a venv own the version your project wants. That rule would have saved me the 30 minutes in 2019 that started this whole series.

## Or just hand this page to your agent

```text
Read https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ and install a newer Python on this machine. First run cat /etc/os-release, python3 --version and ls -l /usr/bin/python* and show me the output, then tell me which method on the page applies and ask me which Python version I want. Never replace /usr/bin/python3; use python-is-python3, uv, deadsnakes or make altinstall as the page describes.
```

The page gives the agent the methods and the rule. Only you know which distro this is and whether the version is for one project or the whole machine.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
