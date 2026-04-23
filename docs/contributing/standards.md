# Code Standards

## Modern Fortran

- Target **Fortran 2003/2008**. Refactor F77/F90 when you touch it.
- Always use `implicit none`; default to `private` and export only what
  callers need.
- Lowercase keywords (`do`, `subroutine`, `module`, `end if`).
- Use the framework kinds (`RKIND`, `R4KIND`, `R8KIND`) — never `real*8`.
- Math variable names can be short (`Ylm`, `Gamma`); routines and modules
  should be fully descriptive.

## Module skeleton

```fortran
module mpas_example
    use mpas_kind_types, only : RKIND
    implicit none
    private

    public :: example_subroutine

contains

    subroutine example_subroutine(input, output)
        real(kind=RKIND), intent(in)  :: input
        real(kind=RKIND), intent(out) :: output

        output = 2.0_RKIND * input
    end subroutine example_subroutine

end module mpas_example
```

## FORD documentation

Public modules and subroutines use the
[FORD](https://github.com/Fortran-FOSS-Programmers/ford/wiki) docstring
format:

```fortran
!***********************************************************************
!
! routine example_subroutine
!
!> \brief One-line summary
!> \details What it computes, key assumptions, references.
!
!-----------------------------------------------------------------------
```

Document the *why*. Cite papers when implementing schemes from the
literature, and note non-obvious constraints (units, valid ranges, side
effects on shared state).

## Magic numbers

Use `mpas_constants` or a clearly-named module-scope parameter:

```fortran
real(kind=RKIND), parameter :: T_FREEZING_K = 273.15_RKIND
```

## Formatting and warnings

Four spaces per indent (matches `core_atmosphere`). No trailing whitespace.
Don't realign existing code in unrelated commits — diff noise hides intent.
Build with `-Wall` (gfortran) or equivalent during development; fix
warnings rather than silencing them.
