
.. DO NOT EDIT.
.. THIS FILE WAS AUTOMATICALLY GENERATED BY SPHINX-GALLERY.
.. TO MAKE CHANGES, EDIT THE SOURCE PYTHON FILE:
.. "gallery/extract-saturation.py"
.. LINE NUMBERS ARE GIVEN BELOW.

.. only:: html

    .. note::
        :class: sphx-glr-download-link-note

        Click :ref:`here <sphx_glr_download_gallery_extract-saturation.py>`
        to download the full example code

.. rst-class:: sphx-glr-example-title

.. _sphx_glr_gallery_extract-saturation.py:


Extract layer saturation
------------------------
Extract layer saturation with Delve.

.. GENERATED FROM PYTHON SOURCE LINES 6-64

.. code-block:: python

    from os import mkdir
    from os.path import exists

    import torch
    from tqdm import trange

    from delve import CheckLayerSat


    class TwoLayerNet(torch.nn.Module):
        def __init__(self, D_in, H, D_out):
            super(TwoLayerNet, self).__init__()
            self.linear1 = torch.nn.Linear(D_in, H)
            self.linear2 = torch.nn.Linear(H, D_out)

        def forward(self, x):
            h_relu = self.linear1(x).clamp(min=0)
            y_pred = self.linear2(h_relu)
            return y_pred

    if not exists("regression/"):
        mkdir("regression/")
    device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
    torch.manual_seed(1)

    for h in [3, 32]:
        # N is batch size; D_in is input dimension;
        # H is hidden dimension; D_out is output dimension.
        N, D_in, H, D_out = 64, 1000, h, 10

        # Create random Tensors to hold inputs and outputs
        x = torch.randn(N, D_in)
        y = torch.randn(N, D_out)

        model = TwoLayerNet(D_in, H, D_out)

        x, y, model = x.to(device), y.to(device), model.to(device)

        layers = [model.linear1, model.linear2]
        stats = CheckLayerSat('regression/h{}'.format(h), save_to="plotcsv", modules=layers, device=device, stats=["lsat", "lsat_eval"])

        loss_fn = torch.nn.MSELoss(reduction='sum')
        optimizer = torch.optim.SGD(model.parameters(), lr=1e-4, momentum=0.9)
        steps_iter = trange(2000, desc='steps', leave=True, position=0)
        steps_iter.write("{:^80}".format(
            "Regression - TwoLayerNet - Hidden layer size {}".format(h)))
        for _ in steps_iter:
            y_pred = model(x)
            loss = loss_fn(y_pred, y)
            steps_iter.set_description('loss=%g' % loss.data)
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

            stats.add_saturations()
        steps_iter.write('\n')
        stats.close()
        steps_iter.close()


.. rst-class:: sphx-glr-timing

   **Total running time of the script:** ( 0 minutes  0.000 seconds)


.. _sphx_glr_download_gallery_extract-saturation.py:


.. only :: html

 .. container:: sphx-glr-footer
    :class: sphx-glr-footer-example



  .. container:: sphx-glr-download sphx-glr-download-python

     :download:`Download Python source code: extract-saturation.py <extract-saturation.py>`



  .. container:: sphx-glr-download sphx-glr-download-jupyter

     :download:`Download Jupyter notebook: extract-saturation.ipynb <extract-saturation.ipynb>`


.. only:: html

 .. rst-class:: sphx-glr-signature

    `Gallery generated by Sphinx-Gallery <https://sphinx-gallery.github.io>`_
