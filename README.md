Model G Particle 2D (proper x–y) — SAFE base + optional vortical motion + quiver-by-speed (updated)

- Updated by Brendan Darrer and ChatGPT 5.4 dated: 12th March 2026 18:47 GMT
- Originally written by Brendan Darrer and ChatGPT 5.2 on date: 27th February 2026 18:43 GMT
- adapted from: @ https://github.com/blue-science/subquantum_kinetics/blob/master/particle.nb and
https://github.com/frostburn/tf2-model-g with https://github.com/frostburn/tf2-model-g/blob/master/docs/overview.pdf
- with ChatGPT 5.4 writing it and Brendan guiding it to produce a clean code.

Install (generally):

    pip3 install numpy scipy matplotlib imageio imageio[ffmpeg]

Run:

    python3 model_g_2d_xy_safe_plus_vort__1b.py --reset --rotseed --quiver --vortlim 0.2

Note for Brendan only:

    python3 -m venv tf-env
    source tf-env/bin/activate

ChatGPT 5.4:

Built from: model_g_2d_xy_safe__2a.py (finite-difference, Dirichlet BC, solve_ivp segmented checkpoints)

Adds:

  - Optional vortical motion (ux,uy evolved) and scalar advection
  - --no_vort to reproduce SAFE behaviour (no velocity, no advection)
  - Fixed (constant) color limits for all 4 panels via --zlim and --vortlim
  - Quiver with constant-length arrows; speed encoded by color

Outputs under out_<run_name>/:

  - frames/frame_XXXX.png
  - <run_name>.mp4
  - checkpoint_2d_plus_vort.npz (auto-resume)
  
=================== UPDATED NOTES====12.3.2026 18:53 =================

Brendan: I have run "model_g_2d_xy_safe_plus_vort__1a.py" (attached) recently (4 to 6 days ago). It doesn't seem to be producing a vortex (in the vortical mode), shown in the 4th plot (for quiver arrows). Can you inspect it to see what the issue is? I attach the video output (MP4) of the plot.

ChatGPT said:

Thought for 5m 2s

Yes — I can see why you’re not “getting a vortex”, even though you’re in VORT mode.

What’s happening (from your MP4 + the code)
In your MP4, the quiver arrows are mostly radial (in/out) rather than circular. That means you do have a velocity field, but it’s mostly irrotational (curl-free), so the vorticity ωz stays near zero except for small edge/shear artifacts.

That matches the code:

If you don’t pass --rotseed, the initial velocity is ux=uy=0.

The velocity evolution is driven mainly by a pressure-gradient term -cs2 * ∇log(rho) (plus viscosity). A gradient force like that does not naturally generate curl from a zero-curl start, so ωz tends to remain ~0.

A real “vortex” (swirl) only appears if you seed rotation (--rotseed) or drive rotation (--omega_drive). Those 

options are in the CLI already.

So: VORT mode ≠ guaranteed vortex. It just means “velocity/advection is enabled”.

Two common gotchas that make it look like “rotseed didn’t work”

You didn’t actually run with --rotseed

Without it, you’ll usually see the radial “sink/source” behaviour, not a swirl.

Checkpoint auto-resume reused an older run

This script loads checkpoint_2d_plus_vort.npz and also tracks which frames were already rendered, so re-running 

with different flags can silently reuse old state/frames.

Fix

A) Quick fix using your existing script

Delete the old output folder (or at least the checkpoint + frames), then run:

    rm -rf out_model_g_2d_xy_safe_plus_vort__1a_vort
    python3 model_g_2d_xy_safe_plus_vort__1a.py --rotseed --quiver --quiver_stride 8 --vortlim 0.2

If the swirl dies too quickly, reduce viscosity or add a drive:

    python3 model_g_2d_xy_safe_plus_vort__1a.py --rotseed --omega_drive 0.5 --nu 0.05 --quiver --vortlim 0.2
    
(--omega_drive is the “keep it spinning” option.)

B) I patched the code to make this easier

I made a new version that adds:

--reset flag: wipes checkpoint/frames/mp4 in the output folder before running

a startup note warning you if VORT mode is on but you forgot --rotseed / --omega_drive

Download: model_g_2d_xy_safe_plus_vort__1b.py (BJD: download issue, it pasted the code for me instead)

Run:
    python3 model_g_2d_xy_safe_plus_vort__1b.py --reset --rotseed --quiver --vortlim 0.2

If you tell me the exact command line you used when generating the MP4 you uploaded, I can confirm 

whether it was (a) missing --rotseed, or (b) a checkpoint reuse case.
