# PCR
## original code from opentron

from opentrons import robot, containers, instruments

source_tubes = containers.load('tube-rack-2ml', 'D2', 'tube rack')
dna_tubes = containers.load('tube-rack-2ml', 'C3', 'dna rack')
output = containers.load('96-PCR-flat', 'C1', 'output')

p20rack = containers.load('tiprack-200ul', 'B2', 'p10-rack')
p200rack = containers.load('tiprack-200ul', 'A1', 'p200-rack')
trash = containers.load('trash-box', 'A3')

p20 = instruments.Pipette(
    trash_container=trash,
    tip_racks=[p20rack],
    min_volume=2,
    max_volume=20,
    axis="a"
)

p200 = instruments.Pipette(
    trash_container=trash,
    tip_racks=[p200rack],
    min_volume=20,
    max_volume=200,
    axis="b"
)

total_volume = 25
DNA_volumes = [1, 2, 3, 4, 5.5, 5]
num_pcr_samples = len(DNA_volumes)
DNA_sources = dna_tubes.wells(0, length=num_pcr_samples)

mix_location = source_tubes.wells('A1')
water_source = source_tubes.wells('C2')

sources = [       #uL per PCR well
    ('B1', 3),    #enzyme -- 4
    ('C1', 2.5),  #buffer -- 5
    ('D1', 2.5),  #dNTP -- 2
    ('A2', 2),    #fprimer -- 3
    ('B2', 2)     #rprimer -- 1
]

sources_total_vol = sum([vol for _, vol in sources])

#Create Master Mix
for name, vol in sources:
    p200.transfer(
        vol * (num_pcr_samples + 1),
        source_tubes.wells(name),
        mix_location)

#Distribute Master Mix
p20.distribute(
    sources_total_vol,
    mix_location,
    output.wells('A1', length=num_pcr_samples))

#Add DNA
p20.transfer(
    DNA_volumes,
    DNA_sources,
    output.wells('A1', length=num_pcr_samples),
    new_tip='always')

#Add water
water_volumes = []
for v in DNA_volumes:
    water_volumes.append(total_volume - v - sources_total_vol)

p20.distribute(
    water_volumes,
    water_source,
    output.wells('A1', length=num_pcr_samples),
    new_tip='always')
